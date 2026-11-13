Bu yazımızda, aşağıdaki yol haritasını kullanarak trendyol kara cuma indirim vakitleri için sistem tasarlayacağız.

![[system-design-tools-tr.svg|700]]

### Fonksiyonel Gereksinimler (Functional Requirements)

- Kullanıcılar ana sayfada flash indirim olan ürünleri görebilir ve arama yapabilir.
- Kullanıcılar seçtikleri ürünleri geçici sepetlerine ekleyebilir.
- Kullanıcılar, dış bir ödeme sağlayıcı (Payment Service Provider) üzerinden sepeti onaylayıp siparişi tamamlayabilir.

### Fonksiyonel Olmayan Gereksinimler (Non-Functional / Scale Metrics)

- Okuma Trafiği (Read-Heavy), saniyede 500.000 istek (Katalog ve Arama)
- Yazma Trafiği (Write-Heavy), saniyede 200.000 istek (Sepet ve Ödeme tetiklenmesi)
- Gecikme (Latency), P99 seviyesinde arama ve listeleme yanıt süresi < 400-500ms olmalı
- Veri Tutarlılığı (Consistency), stok işlemlerinde katı tutarlılık (strict consistency) esastır. Satılan ürünün stokta olmaması (overselling) kabul edilemez.

### API Arayüz

```
// Ürün arama ve listeleme (catalog service)
GET /products/search?q={term}&catalog_id={id}&limit={count} ==> Product[]

// Sepete ürün ekleme (cart service - Redis)
Headers: bearer {token}
POST /carts/items
{
	product_id,
	quantity
} ==> 201, Cart

// Sipariş tamamlama (order service)
POST /orders/checkout
Headers: Idempotency-Key
{
	cart_id,
	payment_method,
	shipping_address_id
} ==> 201, Order
```

### Veri Modeli

```
Product
- id
- category_id
- name
- description
- price
- ...

Cart
- id
- user_id
- product_id
- quantity
- ...
  
Inventory
- product_id
- quantity

Order
- id
- user_id
- product_model[]
- payment_details
- ...

order_outbox
- order_id
- status (completed, pending)
```

### Yüksek Seviye Mimari

Aşağıda uçtan uca mimari tasarım çizilmiştir.


![[trendyol-system-design.svg]]

### Darboğaz Analizi ve Ölçekleme

#### Kara Cuma Sipariş ve Stok Yönetimi

Kabaca düşünürsek, tek bir ilişkisel veritabanı (PostgreSQL gibi) yazma ağırlıklı yükte saniyede on binler mertebesinde işlem karşılayabilir. Redis ise tek bir instance’ta yüz binler mertebesine çıkabilir. İlk darboğaz senaryomuz şu olsun: Kara Cuma vakti saniyede 200 bin istekle sepet onayı ve ödeme akışı tetikleniyor.

Her başarılı ödemeden sonra stoğu doğrudan PostgreSQL’de düşürürsek, kampanyalı popüler bir ürünün stok satırı üzerinde yoğun bir row-level lock yarışı başlar. Diğer işlemler kuyrukta bekler, connection pool hızla tükenir ve CPU tavan yapar.

**Çözüm: Stoğu Redis’te pre-allocate etmek.** Kampanya başlamadan önce stok miktarlarını Redis’e yükleriz. Kullanıcı checkout’a geçtiğinde, Redis üzerinde bir Lua script ile atomik olarak stok düşülür ve kullanıcı için rezervasyon (soft lock) oluşturulur. Stok yoksa veya rezerv edilmiş ise istek veritabanına hiç ulaşmadan reddedilir. Yükün büyük kısmını Redis karşılar, veritabanı korunur.

Redis’teki rezerv geçici bir kilittir. Her rezervi, kilidin yanında bitiş zamanıyla birlikte bir deftere (sorted set) yazarız. Stoğu geri verme yetkisi, bu defterden kaydı silmeyi başaran tek tarafa aittir. Ödeme başarısız ise rezervasyon silinir ve stok geri artırılır. Bir temizlik job’ı defteri periyodik tarayıp süresi dolanların stoğunu geri verir. Ödeme başarılıysa kayıt silinir ama stok artırılmaz; sipariş veritabanına yazılarak rezerv kalıcı hale gelir. Süre dolduktan sonra ödeme gelirse kayıt zaten yoktur, sipariş yazılmaz ve ödeme iade edilir. Kaydı yalnızca bir taraf silebildiği için stok ne iki kez geri artar ne de eksik kalır. Bu yöntem sayesinde, veritabanına sadece stoğu ayırabilmiş ve ödemeyi yapmış kullanıcıların sipariş kayıtları yazılır.

#### Dikkat Edilmesi Gerekenler

Redis ve Lua script’leri akışı hızlı ve atomik yapar, ama tek başlarına yetmez. Kara Cuma gibi yoğun günlerde sistemi ayakta tutan birkaç nokta var. Ödeme süresini rezerv süresinden kısa tutmak (örneğin ödeme için 5, rezerv için 10 dakika) “ödeme alındı ama rezerv düştü” durumunu nadir bir istisnaya çevirir; yine de olursa order yazılmaz, ödeme iade edilir. Her checkout isteği benzersiz bir idempotency key taşımalı, API Gateway aynı key’in ikinci kez gelmesini `409 Conflict` ile reddetmelidir; böylece çift order oluşmaz. Redis hızlı bir kapıdır ama tek doğru kaynak değildir, bu yüzden son söz her zaman veritabanında olmalı ve order yazılırken veritabanında da stok kontrolü yapılmalıdır. Son olarak Redis çökerse sayaçlar kaybolabileceği için, sayaçları PostgreSQL’deki stok ve onaylı order’lardan yeniden hesaplayabilecek bir yol bulunmalıdır.

#### Saniyede 500K okuma yapabilmek

Gelen okuma isteklerini `catalog-service` üzerinden doğrudan veritabanına yönlendirirsek, özellikle arama (search) özelliği çok hızlı bir şekilde darboğaza girer ve sistem çökebilir. Bu yükü azaltmak için birkaç katmanlı bir yaklaşım izleyebiliriz:

1. **Arama için Elasticsearch:** Arama sorgularını PostgreSQL yerine araya koyacağımız Elasticsearch’e yönlendirerek veritabanını bu yükten kurtarabiliriz.
2. **Redis Cluster ile önbellekleme:** Ürün görseli ve ürün detayı gibi bilgiler sık değişmediği için bunları cluster yapısında çalışan Redis üzerinde önbelleğe alabiliriz. Böylece okuma isteklerinin büyük kısmı veritabanına hiç ulaşmaz.
3. **CDN ile uç noktada sunum:** Sadece arama değil, ana sayfayı da düşünmemiz gerekiyor. Ana sayfada çoğu kullanıcıya genellikle aynı ürünler ve indirimler gösterildiği için bu içeriği CDN üzerinden sunabiliriz. Bu sayede istekler uygulamamıza bile gelmeden karşılanır.

#### Hot Key Problemi

Kara Cuma’da herkes aynı birkaç popüler ürünü ya da aramayı sorgular. Redis cache’te bu ürünlerin key’i tek bir shard’da durduğu için tüm istekler aynı noktaya yığılır. Buna hot key problemi denir ve o shard’ı darboğaza çevirir. Çözümlerden biri, aynı veriyi tek key yerine `productId:0` ile `productId:9` arası sabit sayıda kopya olarak tutmaktır. Her istek rastgele bir kopyayı okur, kopyalar farklı shard’lara düştüğü için yük yayılır. Bunun bedeli, her kopyanın ayrı güncellenmesi ve kopya sayısı arttıkça cache hit oranının düşmesidir; bu yüzden kopya sayısını sabit ve düşük tutmak gerekir.
#### Redis Lua Script ile Atomik İşlemler

Redis single-threaded'dir ve bir lua script'i çalışırken başka hiçbir komut araya giremez (atomicity). 

Stok düşürme (rezerv alma)

```lua
-- KEYS[1] = "inventory:{1234}"     (stok sayacı)
-- KEYS[2] = "reservations:{1234}"  (defter, sorted set)
-- ARGV[1] = "userId:quantity"      (defter kaydı)
-- ARGV[2] = quantity
-- ARGV[3] = bitiş zamanı (unix timestamp)
local stock = tonumber(redis.call('GET', KEYS[1]))

if stock and stock >= tonumber(ARGV[2]) then
  redis.call('DECRBY', KEYS[1], ARGV[2])
  redis.call('ZADD', KEYS[2], ARGV[3], ARGV[1])
  return 1 -- Başarılı, stok rezerve edildi (soft-lock)
else
  return 0 -- Başarısız, stok yok
end
```

Stok artırma (rezervi serbest bırakma)

```lua
-- KEYS[1] = "inventory:{1234}"
-- KEYS[2] = "reservations:{1234}"
-- ARGV[1] = "userId:quantity"
-- ARGV[2] = quantity
if redis.call('ZREM', KEYS[2], ARGV[1]) == 1 then
  redis.call('INCRBY', KEYS[1], ARGV[2])
  return 1 -- Rezerv vardı, stok geri verildi
else
  return 0 -- Kayıt yok: başkası zaten halletti, bir şey yapma
end
```

Stok düşürürken rezervi bir deftere (sorted set) yazarız. Stoğu geri artırma işini yapan script önce bu rezervi silmeyi dener; yalnızca silme başarılı olursa stoğu artırır. Kayıt zaten yoksa, başka biri iade etmiş demektir ve stoğa dokunmayız. Hepsi tek bir Lua script’inde atomik çalıştığı için aynı iade iki kez gelse bile stok bir kez artar.
#### Change Data Capture (CDC)

PostgreSQL’de insert ve update işlemleri, her seferinde veri dosyasına (disk) yazmak yerine önce WAL’e (Write-Ahead Log) yazıldığı için hızlıdır. WAL’e yazma sıralı (sequential) bir ekleme işlemidir ve rastgele disk erişiminden çok daha ucuzdur. Değişiklik önce bellekte uygulanır ve WAL’e kaydedilir, ilgili veri dosyalarına yazma işini ise PostgreSQL’in arka plandaki mekanizmaları (checkpointer, background writer) daha sonra yapar. Bu log aynı zamanda veritabanındaki her değişikliğin sıralı bir kaydını tuttuğu için dışarıya stream edilebilir.

Veritabanındaki değişiklikleri neredeyse anlık yakalama yaklaşımına **CDC** denir. PostgreSQL’de `wal_level = logical` ayarı yapılarak WAL kayıtları okunabilir hale getirilir. **Debezium**, bir logical replication client olarak bu kayıtları okur, standart bir olay formatına çevirir ve Kafka’ya aktarır. Bir consumer ile bu yapı, Elasticsearch’i PostgreSQL ile senkron tutmak için kullanılabilir.

Peki, veriyi veritabanına yazdıktan sonra herhangi bir problemden dolayı veri değişikliği (event) Kafka’ya asenkron olarak gönderilemezse ne olur? İşte tam bu noktada **Outbox Pattern** devreye girer. Uygulama, veritabanını güncellediği transaction içerisinde aynı zamanda outbox tablosuna da ilgili olayı yazar. Böylece veri ile event ya ikisi birlikte kaydedilir ya da hiç kaydedilmez. Debezium, bu tabloyu veritabanının değişiklik günlüğü (WAL - Write-Ahead Log) üzerinden okuyarak olayları Kafka’ya iletir. Debezium bir süre çalışmasa bile kaldığı noktayı hatırlar ve tekrar ayağa kalktığında kaldığı yerden devam eder; böylece hiçbir olay kaybolmaz. Olaylar "en az bir kez" (at-least-once) iletildiği için consumer’lar aynı olayı tekrar alma ihtimaline karşı **idempotent** olacak şekilde tasarlanmalıdır.

![[trendyol-cdc.svg]]

Mimaride bu yöntem, fatura-bildirim servisleri ve elastic search için kullanıldı.

### Sonuç

Bu yazımızda, Kara Cuma gibi saniyede yüzbinlerce isteğin aktığı devasa kampanyalarda bir e-ticaret altyapısının nasıl ayakta kalabileceğini ele aldık. Yüksek yazma trafiğindeki veritabanı kilitlenmelerini Redis ve Lua script tabanlı soft-lock mekanizmalarıyla aşarken; okuma yükünü Elasticsearch, Redis Cluster ve CDN katmanlarıyla güvenle dağıttık. Ayrıca CDC, Debezium ve Outbox Pattern gibi yaklaşımlarla sistem genelinde veri tutarlılığını ve güvenilir asenkron akışı güvence altına aldık. Elbette teoride kurgulanan her mimari gibi, bu yapının da canlı trafik öncesinde kapsamlı yük ve stres testleriyle doğrulanıp ince ayarlarının yapılması büyük önem taşıyor.

Diğer blog yazılarında görüşmek üzere!

### Referanslar

[5 büyük sistem tasarımı](https://mvpakademi.com/course/5-buyuk-sistem-tasarimi)