Bu yazıda derin dizi modelleme konusuna göz atacağız.

Size bir cisim verilip, hangi yöne gidiyor diye sorulduğunda, net bir şey söylemek zordur. Fakat cismin önceki konumları hakkında da bilgi sahibiysek cisme yön tayin etmek kolaylaşacaktır.

![[ball-prediction.svg]]

Sinir ağlarının yapısını hatırlayalım.

![[feed-forward-network-revisit.svg]]

Burada $x_t \in \mathbb{R}^m$ ve $\hat{y}_t \in \mathbb{R}^n$ dir. $t$ zamanı temsil etsin. O zaman $t=0,1,2,...$ şeklinde sıralı bir girdi çıktı dizisi elde ederiz. Şöyle ki,

![[time-step.svg]]

Sadece zaman dilimlerine ayırıp $f(x_t)=\hat{y}_t$ şeklinde konumlandırırsak mimariyi, zamana göre sıralı olan girdi ve çıktılar arasında bir ilişki kurmamış oluruz. Başta verdiğimiz cisim örneği gibi, geçmişten gelen çıktıları da o anki işlenen girdiye dahil etmek istiyoruz. Böylece anlamlı bir çıktı elde edelim işlem sonunda.

![[time-step-relation.svg]]

Şimdi yeni bir parametre tanıtıyoruz mimariye $h_t$. Yeni girdi çıktı formülü şöyle ki,

$$
\hat{y}_t=f(x_t, h_{t-1})
$$
burada $t$ anındaki, girdi $x_t$, geçmiş hafıza $h_{t-1}$, çıktı ise $\hat{y}_t$ dir. Sıra geldi genelleştirmeye, her $t$ anındaki girdi çıktı hesabı için aynı katmanı kullanacağız. Çizimlerde gördüğünüz katmanlar aslında aynı katman fakat zamana bağlı olarak girdi ve çıktılar farklıydı.

### Tekrarlayan Sinir Ağları

![[neurons-with-recurrence.svg]]

Formül olarak,

$$
h_t=f_W(x_t,h_{t-1})
$$
Burada, $h_t$ katman durumunu, $f_W$ ağırlıkların fonksiyonunu, $x_t$ girdi vektörünü ve $h_{t-1}$ eski katman durumunu temsil eder. Her $t$ zaman adımında $h_t$ güncellenir.

```python
my_rnn = RNN()
hidden_state = [0,0,0,0]

sentence = ["I", "love", "recurrent", "neural"]

for word in sentence:
	prediction, hidden_state = my_rnn(word, hidden_state)

next_word_prediction = prediction
# >>> "networks!"
```

### RNN Durum Güncelleme ve Çıktı

Durum güncelleme formülü,

$$
h_t=tanh(W^T_{hh}h_{t-1}+W^T_{xh}x_t)
$$
çıktı,

$$
\hat{y}_t=W^T_{hy}h_t
$$

Her adımda aynı ağırlık matrisleri kullanılır.

`TensorFlow` ile tekrarlayan sinir ağı tasarlayalım.

```python
class MyRNNCell(tf.keras.layers.Layer):
	def __init__(self, rnn_units, input_dim, output_dim):
		super(MyRNNCell,self).__init__()
		
		# initialize weight matrices
		self.W_xh = self.add_weight([rnn_units, input_dim])
		self.W_hh = self.add_weight([rnn_units, rnn_units])
		self.W_hy = self.add_weight([output_dim, rnn_units])
		
		# initialzie hidden state to zeros
		self.h = tf.zeros([rnn_units, 1])
		
	def call(self, x):
		# update the hidden state
		self.h = tf.math.tanh(self.W_hh * self.h + self.W_xh * x)
		
		# compute the output
		output = self.W_hy * self.h
		
		# return the current output and hidden state
		return output, self.h
```

Hazır kütüphane ile RNN kuralım,

```python
from tf.keras.layers import SimpleRNN

model = SimpleRNN(rnn_units)

from torch.nn import RNN

model = RNN(input_size, rnn_units)
```

### Tasarım Kriterleri

- Verilerin veya cümlelerin farklı uzunluklarda olabilmesi ile başa çıkmak.
- Dzideki uzak geçmişte yer alan ögeler ile güncel ögeler arasındaki ilişki ve bağımlılıkları koruyabilmek.
- Ögelerin dizilim sırasını kaybetmemek
- Dizinin farklı konumlarında aynı parametreleri kullanmak.

### Bir Sonraki Kelimeyi Tahmin Etmek

![[next-word.svg]]

Sinir ağı için bir kodlama diline ihtiyacımız var. Çünkü ağ sayısal veriler kabul etmektedir.

#### Gömme

![[embedding.svg]]

#### Değişken Dizi Uzunluğunu Ele Almak

![[next-words-multi.svg]]

#### Model Uzun Süreli Bağımlılıklar

"**France** is where I grew up, but I now live in Boston. I speak fluent _____."

Cümlenin tamamlanması için, uzak geçmişten bilgiye ihtiyaç vardır.

#### Sıra Değişimini Yakalama

"The food was good, not bad at all."

"The food was bad, not good at all."

### Zaman İçinde Geri Yayılım

![[loss.svg]]

$h_0$ a göre gradyan hesablanır.

### RNN Uygulama ve Limitler

- Kodlama darboğazı, girdi boyutu sabit olmamasına karşın, sabit boyutlu bir vektöre gömme yapılır ve veri kaybına uğranır.
- Yavaş ve paralelleştirme yok.
- Uzun vadeli bağımlılıkları öğrenememe.

### Tek ihtiyacınız Olan Dikkat -- Attention is All You Need

- Pozisyon bilgisini kodla.
- Sorgu, anahtar, değer bilgisini çıkar.
- Dikkat matrisini hesapla.
- Özellikleri çıkar yüksek dikkat ile.

Dikkat skoru, sorgu ve anahtar arasındaki benzerlik miktarı kadardır. Kosinus benzerliği olarak adlandırılır.