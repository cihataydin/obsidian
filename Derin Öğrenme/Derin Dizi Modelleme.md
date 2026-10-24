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
