### Giriş

Sistem tasarımı sırasında aşağıdaki mantık sırasını kullanarak ilerleyeceğiz.

![[system-design-tools.svg]]

### Sistem Tanımı

Çeşitli etkinliklerin yer aldığı, kullanıcıların bu etkinlikler için bilet veya biletler satın alabildiği sistem.

### Fonksiyonel Gereksinimler

- Kullanıcılar gelecek etkinlikleri görüntüleyebilir olmalı.
- Bu etkinlikler için bilet satın alma işlemi yapabilir olmalı.
- Etkinlikleri arayabiliyor olmalı.

### Fonksiyonel Olmayan Gereksinimler

- Low latency search (düşük gecikmeli arama) < 500ms
- Okuma işlemlerimiz yazma işlemlerimizden daha çok (Okuma > Yazma)
- Yüksek talep gören etkinlikleri yönetme.
- Arama yapma, `Availability` (Erişilebilirlik), bilet satın almada, `Consistency` (Tutarlılık)

### API Arayüz

```
GET /events/{eventId} => etkinlik, mekan, sanatçı, biletler[]

GET /search?keyword={keyword}&startDate={sDate}&endDate={eDate}&pageSize{}&pageNumber={} => etkinlik[]

POST /bookings/{eventId}
{
	ticketIds: [],
	paymentDetails: {}
}
```

### Veri Modeli

```
events
- id
- venue_id
- performer_id
- tickets[]
- name
- description
- ...

venues
- id
- location
- seat_map

performers
- id
- ...

tickets
- id
- event_id
- seat
- price
- status {available, booked}
- user_id
  
bookings
- user_id
- tickets[]
```

### Yüksek Seviye Mimari

![[biletix-system-design.svg]]

### Darboğaz Analizi ve Ölçekleme

Örnek bir senaryo düşünelim. Tarkan konser biletleri satışa çıksın ve en öndeki bir koltuğa aynı milisaniye içinde binlerce istek gelsin. Eğer bu yükü doğrudan veritabanına gönderip satır kilitleme (`row-level lock`) yapılırsa, veritabanının `CPU`'su anında tavan yapar ve tüm sistem kilitlenir. Bu yüzden, koltuğun müsaitlik durumunu ve kilitleme işini veritabanından önce `RAM` üzerinde, yani `Redis`'te çözülmesi gerekir. `Redis`'in tek thread (`single-threaded`) yapısı ve `SetNX` (Set If Not Exists) komutu ihtiyacı çözer. Burada önemli nokta `TTL(Time to Live)` kullanılması, böylece müşteri süre aşımına uğrasa dahi aynı bilet tekrar satın alınabilir statüde olabilecektir. Bu basit mantık sayesinde, yarış durumu (race condition) olmadan yüz binlerce istek `RAM` üzerinde eritilir.

Yine Tarkan konseri biletleri satışa çıksın. Cumartesi saat 10:00 olsun. Milyonlarca kullanıcı aynı anda sayfayı yenilesin. Eğer bu isteklere doğrudan cevap verilirse, aradaki redis ve load balancer fayda göstermeden, arka taraftaki servis anlık (spike) yükten dolayı boğulur. Kullanıcılar 503 hatası almaya başlar. Bir sanal bekleme odası (virtual waiting room) kullanılır. Sistemin kapasitesi kadar kullanıcı içeri alınır diğer kullanıcılar ise bekletilir. Örnek olarak bir `API Gateway` ve `Queue` ikilisi düşünülebilir. Sistem kapasitesi müsait oldukça `SSE (Server Sent Events)` gönderilerek bekleme odasındaki kullanıcılar içeriye davet edilir.

Queue'daki mantık,
- Sıra numarası ve benzersiz bir `token` verilir.
- Kullanıcılar kuyruğa alınır.
- Kapasite kadar kullanıcı içeri alınır. (`throttling`)

Bu mimari sayesinde, sistem (core backend, veritabanı) sadece hizmet verebileceği kadar insanla muhatap olur.

### Referanslar

[5 büyük sistem tasarımı](https://mvpakademi.com/course/5-buyuk-sistem-tasarimi)
