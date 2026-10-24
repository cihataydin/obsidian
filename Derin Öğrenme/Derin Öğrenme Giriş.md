Bu yazıda derin öğrenme temellerine göz atacağız.

### Algılayıcı - İleri Yayılma

![[the-perceptron-forward-propagation.svg]]


$$
\hat{y} = g\left(w_0 + \sum_{i=1}^{m} x_i w_i\right)
$$
$\hat{y}$ `çıktı`, $g$ `aktivasyon foksiyonu`, $x_i$'ler `girdiler`, $w_i$'ler `ağırlıklar`, $w_0$ `bias` olarak adlandırılır. Ağırlıkların ve girdilerin lineer kombinasyonu, lineer olmayan bir fonksiyondan geçirilerek bir çıktı üretilir. Bu yapıya **ileri yayılım** gösteren bir **algılayıcı** denir. Girdiler ve ağırlıklar vektör/matris şeklinde yazılır ise, aşağıdaki eşitlik elde edilir.

$$
\hat{y} = g(w_0 + \mathbf{x}^T \mathbf{w})
$$
Burada $\mathbf{x}$ ve $\mathbf{w}$ şöyle ki,
$$
\mathbf{x} = \begin{bmatrix} x_1 \\ x_2 \\ \vdots \\ x_m \end{bmatrix}, \quad \mathbf{w} = \begin{bmatrix} w_1 \\ w_2 \\ \vdots \\ w_m \end{bmatrix}
$$

### Aktivasyon Fonksiyonları

Günlük hayattaki problemler sadece doğrusal (lineer) işlemlerle çözülemeyecek kadar karmaşıktır. Bu yüzden sinir ağlarında gerçek dünyadaki doğrusal olmayan ilişkileri öğretebilmek adına **aktivasyon fonksiyonları** kullanılır.

#### Sigmoid

$0$ ve $1$ aralığında değer alır.

$$
g(z)=\sigma(z) = \frac{1}{1 + e^{-z}}
$$

Türevi,

$$
g^{'}(z)=g(z)(1-g(z))
$$

`TensorFlow` ve `PyTorch` kullanımları,

```python
tf.math.sigmoid(z)
torch.sigmoid(z)
```
#### Hiperbolik Tanjant

$-1$ ile $1$ aralığında değer alır.

$$
g(z)=\tanh(z) = \frac{e^z - e^{-z}}{e^z + e^{-z}}
$$

Türevi,

$$
g^{'}(z)=1-g^{2}(z)
$$
`TensorFlow` ve `PyTorch` kullanımı,

```python
tf.math.tanh(z)
torch.tanh(z)
```

#### Rectified Linear Unit (ReLU)

`0` dan büyük değerler alır.

$$
g(z)=max(0,z)
$$

Türevi,

$$
g^{'}(z) = \begin{cases} 
    1, & z > 0 \\ 
    0, & otherwise
\end{cases}
$$
`TensorFlow` ve `PyTorch` kullanımı,

```python
tf.nn.relu(z)
torch.nn.ReLU(z)
```

### Sinir Ağları İnşası

#### Algılayıcı - Çoklu Çıktı

Birden fazla algılayıcı kullanılarak çoklu çıktıya sahip bir katman inşa edilebilir.

![[multi-output-perceptron.svg]]

$z_i$ aşağıda tanımlanmıştır. $(i=1,2)$

$$
z_i = w_{0,i} + \sum_{j=1}^{m} x_j w_{j,i}
$$
`TensorFlow` kullanımı,


```python
import tensorflow as tf

layer = tf.keras.layers.Dense(units=2)
```

`PyTorch` kullanımı

```python
import torch.nn as nn

layer = nn.Linear(in_features=m, out_features=2)
```

#### Yoğun Katman

Her girdinin, her çıktıya bağlı olduğu katmanlara **yoğun katman** denir. Aşağıda örnek yoğun katman tasarımı bulunmaktadır.

`TensorFlow` kullanımı,

```python
class MyDenseLayer(tf.keras.layers.Layer):
	def __init__(self, input_dim, output_dim):
		super(MyDenseLayer, self).__init__()
		
		# Initialize weights and bias
		self.W = self.add_weight([input_dim, output_dim])
		self.b = self.add_weight([1, output_dim])
	
	def call(self, inputs):
		# Forward propagate the inputs
		z = tf.matmul(inputs, self.W) + self.b
		
		# Feed through a non-linear activation
		output = tf.math.sigmoid(z)
		return output
```

`PyTorch` kullanımı,

```python
class MyDenseLayer(nn.Module):
	def __init__(self, input_dim, output_dim):
		super(MyDenseLayer, self).__init__()
		
		# Initialize weights and bias
		self.W = nn.Parameter(torch.randn(input_dim, output_dim, requires_grad=True))
		self.b = nn.Parameter(torch.randn(input_dim, output_dim, requires_grad=True))
	def forward(self, inputs):	
		# Forward propagate the inputs
		z = torch.matmul(inputs, self.W) + self.b
		
		# Feed through a non-linear activation
		output = torch.sigmoid(z)
		return output
```

#### Tek Katmanlı Sinir Ağı
Şu ana kadar sadece girdi ve çıktı katmanı vardı. Bir de gizli katman ekleyelim. Bu katmanın elemanları da yine bir algılayıcı olarak görev yapmaktadır.

![[single-layer-neural-network.svg]]

Gizli katman çıktıları,

$$
z_i = w_{0,i}^{1} + \sum_{j=1}^{m} x_j w_{j,i}^{1}
$$

Son çıktılar,

$$
\hat{y}_i = g(w_{0,i}^{2} + \sum_{j=1}^{n} g(z_j) w_{j,i}^{2})
$$

`TensorFlow` kullanımı,

```python
import tensorflow as tf

model = tf.keras.Sequential([
	tf.keras.layers.Dense(n),
	tf.keras.layers.Dense(2)
])
```

`PyTorch` kullanımı,

```python
from torch import nn

model = nn.Sequential(
	nn.Linear(m, n),
	nn.ReLU(),
	nn.Linear(n, 2)
)
```

#### Derin Sinir Ağı

İşte mevzunun koptuğu noktadayız! Artık gizli katman sayısını istediğimiz kadar artırabiliriz. Her katman bir önceki katmandan aldığı çıktıyı işler ve algılayıcılar üzerinden yeni bir girdi-çıktı hesabı yaparak bilgiyi derinleştirir.
![[deep-neural-network.svg]]

Gizli katman çıktıları,

$$
z_{k,i} = w_{0,i}^{k} + \sum_{j=1}^{n_{k-1}} g(z_{k-1, j}) w_{j,i}^{k}
$$

`TensorFlow` kullanımı,

```python
import tensorflow as tf

model = tf.keras.Sequential([
	tf.keras.layers.Dense(n1),
	tf.keras.layers.Dense(n2),
	#...
	tf.keras.layers.Dense(2)
])
```

`PyTorch` kullanımı,

```python
from torch import nn

model = nn.Sequential(
	nn.Linear(m, n1),
	nn.ReLU(),
	#...
	nn.ReLU(),
	nn.Linear(nk, 2)
)
```

### Kayıp Belirleme

Tahmin edilen ve aslında gerçekleşen değer arasındaki farka **kayıp** denir.

$$
\mathcal{L}(f(x^{i};W), y^{i})
$$
İlk bileşen tahmin edilen değer, ikincisi aslında gerçekleşen değerdir. $\mathcal{L}$ bu iki değer arasındaki farkı ölçen kaybı belirleme fonksiyonudur.

#### Gözlemlenen Kayıp

$J$ **Maliyet fonksiyonu** olarak adlandırılır. Sinir ağının yaptığı tahminin ne kadar doğru veya yanlış olduğunu ölçmek için kullanılır.

$$
J(\mathbf{W}) = \frac{1}{n} \sum_{i=1}^{n} \mathcal{L}\left( f(x^{(i)}; \mathbf{W}), y^{(i)} \right)
$$

#### İkili Çapraz Entropi Kaybı

Eğer çıktılar ikili değer `{0,1}` ile sınırlı ise yani ikili sınıflandırma problemi üzerine çıktı üretilecek ise aşağıdaki maliyet fonksiyonu kullanılır. Örneğin, müşteri ürünü satın alacak mı veya almayacak mı gibi problemler düşünülebilir.

$$
J(\mathbf{W}) = -\frac{1}{n} \sum_{i=1}^{n} \left[ y^{(i)} \log(f(x^{(i)}; \mathbf{W})) + (1 - y^{(i)}) \log(1 - f(x^{(i)}; \mathbf{W})) \right]
$$

`TensorFlow` ile kullanımı,

```python
loss = tf.reduce_mean(tf.nn.softmax_cross_entropy_with_logits(y, predicted))
```

`PyTorch` kullanımı,

```python
loss = torch.nn.functional.cross_entropy(predicted, y)
```

#### Ortalama Karesel Hata

Çıktıları reel sayılar olan bir sinir ağında maliyeti hesaplamak için, aşağıdaki fonksiyon kullanılır.

$$
J(\mathbf{W}) = \frac{1}{n} \sum_{i=1}^{n} \left( y^{(i)} - f(x^{(i)}; \mathbf{W}) \right)^2
$$

`TensorFlow` kullanımı,

```python
loss = tf.reduce_mean(tf.square(tf.subtract(y, predicted)))
loss = tf.keras.losses.MSE(y, predicted)
```

`PyTorch` kullanımı,

```python
loss = torch.nn.functional.mse_loss(predicted, y)
```

### Sinir Ağlarını Eğitmek

Temel amaç, kayıp değerini olabilecek en düşük seviyeye çekmektir. Bunu başarabilmek için sinir ağındaki parametreleri, yani **ağırlıkları** sürekli olarak yeniden düzenleriz. Hedefimiz, kaybın minimum olduğu o en ideal nokta için doğru ağırlık değerlerini bulmaktır.
$$
\boldsymbol{W}^* = \arg\min_{\boldsymbol{W}} \frac{1}{n} \sum_{i=1}^{n} \mathcal{L}\left(f(x^{(i)}; \boldsymbol{W}), y^{(i)}\right)
$$

$$
\boldsymbol{W}^* = \arg\min_{\boldsymbol{W}} J(\boldsymbol{W})
$$

Burada, $\boldsymbol{W} = \{\boldsymbol{W}^0, \boldsymbol{W}^1, ...\}$ ağırlık matrislerini temsil eder.

### Gradyan İnişi

Ağırlıkların yeniden hesaplanması, gradyan inişi yöntemi ile hesaplanır.

Algoritma,
- Ağırlıklara rastgele atama yap  ~ $\mathcal{N}(0, \sigma^2)$ 
- Yakınsayana kadar döngü kur:
	- `gradyan` hesapla, $\frac{\partial J(\boldsymbol{W})}{\partial \boldsymbol{W}}$ 
	- Ağırlıkları güncelle, $\boldsymbol{W} \leftarrow \boldsymbol{W} - \eta \frac{\partial J(\mathbf{W})}{\partial \mathbf{W}}$
- Ağırlıkları dön.

Burada $\eta$ öğrenme oranıdır.

`TensorFlow` kullanımı,

```python
import tensorflow as tf

weights = tf.Variable([tf.random.normal()])

while True: #loop forever
	with tf.GradientType() as g:
		loss = compute_loss(weights)
		gradient = g.gradient(loss, weights)
	
	weights = weights - lr * gradient
```

`PyTorch` kullanımı,

```python
import torch

weights = torch.randn(1, requires_grad=True)

while True:
  loss = compute_loss(weights)

  loss.backward()

  with torch.no_grad():
    weights -= lr * weights.grad
    weights.grad.zero_()
```
### Gradyan Hesaplama - Geri Yayılım

Ağırlıklardaki küçük bir değişikliğin $J(\boldsymbol{W})$ üzerindeki etkisini araştıralım.

![[gradient-backpropagation.svg]]

$$
\frac{\partial J(\mathbf{W})}{\partial w_2} = \frac{\partial J(\mathbf{W})}{\partial \hat{y}} * \frac{\partial \hat{y}}{\partial w_2}
$$

Burada $w_2$ ağırlığının $J(\mathbf{W})$ üzerindeki etkisini zincir kuralını kullanarak tanımladık.

$$
\frac{\partial J(\mathbf{W})}{\partial w_1} = \frac{\partial J(\mathbf{W})}{\partial \hat{y}} * \frac{\partial \hat{y}}{\partial w_1}
$$
$\hat{y}$ fonksiyonu dolaylı olarak $z_1$ fonksiyonu üzerinden $w_1$ ağırlığına bağlıdır. Yine zincir kuralı uygularsak,

$$
\frac{\partial J(\mathbf{W})}{\partial w_1} = \frac{\partial J(\mathbf{W})}{\partial \hat{y}} * \frac{\partial \hat{y}}{\partial z_1} * \frac{\partial z_1}{\partial w_1}
$$
elde ederiz. Böylece $J(\mathbf{W})$ maliyet fonksiyonuna $w_1$ ağırlığının da etkisini belirledik. Maliyet fonksiyonunu minimize etmek için, bütün ağırlıklara göre değişimi hesaplayıp, yeni ağırlıklar atayabiliriz.

### Maliyet Fonksiyonu Optimizasyon

Ağırlıkların güncellenme adımında,

$$
\boldsymbol{W} \leftarrow \boldsymbol{W} - \eta \frac{\partial J(\mathbf{W})}{\partial \mathbf{W}}
$$

eğer $\eta$ gerekenden küçük bir değer alırsa, maliyet fonksiyonu yavaş yakınsar ve yanlış lokal minimumda takılı kalabilir. Gerekenden büyük bir değer alırsa, maliyet fonksiyonu ıraksar. Stabil bir değerde ise, sorunsuz bir şekilde yakınsar ve lokal minimum değerleri ihmal eder. Bu sebeplerden dolayı, öğrenme oranı dinamik ve adapte olabilen bir değer olarak kurgulamalıdır. Sonuç olarak $\eta$, gradyan büyüklüğü, öğrenme hızı, ağırlıklar boyutu gibi parametrelere göre hesaplanmalıdır. 

### Gradyan İniş Algoritmaları

```python
tf.keras.optimizers.SGD
tf.keras.optimizers.Adam
tf.keras.optimizers.Adadelta
tf.keras.optimizers.Adagrad
tf.keras.optimizers.RMSProp

torch.optim.SDG
torch.optim.Adam
torch.optim.Adadelta
torch.optim.Adagrad
torch.optim.RMSProp
```

### Parçaları Birleştirelim

```python
import tensorflow as tf

model = tf.keras.Sequential([...])

# pick your favorite optimizer
optimizer = tf.keras.optimizer.SDG()

while True: # lop forever
	# forward pass through the network
	prediction = model(x)
	
	with tf.GradientTape() as tape:
		# compute the loss
		loss = compute_loss(y, prediction)
		
	# update the weights using the gradients
	grads = tape.gradient(loss, model.trainable_variables)
	optimezer.apply_gradients(zip(grads, model.trainable_variables))
```

### Mini Yığın

Büyük veri setlerinde tüm veri üzerinden $\frac{\partial J(\mathbf{W})}{\partial \mathbf{W}}$ türevini hesaplamak maliyetli ve bazen imkansızdır. Bu zorluğu aşmak için veri seti **küçük alt kümelere** bölünür. Maliyet fonksiyonunun türevi, tüm veri kümesi yerine her defasında sadece bu küçük gruplar için hesaplanarak model ağırlıkları güncellenir.

$$
\frac{\partial J(\mathbf{W})}{\partial \mathbf{W}} = \frac{1}{B} \sum_{k=1}^{B} \frac{\partial J_k(\mathbf{W})}{\partial \mathbf{W}}
$$
Ağ eğitimi sırasında, sorunsuz yakınsama sağlar ve daha büyük öğrenme oranı kullanımına izin verir. Ayrıca mini yığın yöntemi paralel hesaplama ile hızlı eğitime de yol açar.

### Aşırı Uyum

Makine öğrenmesinde ideal dengeyi yakalamak kritik önem taşır. Eğer **aşırı uyum** olursa; model veriyi genelleştiremez, aşırı kompleks bir hâl alır ve gereğinden fazla parametre kullanarak gürültüyü ezberler. Tam aksine **yetersiz uyum** gösterirse; model verideki kalıpları öğrenecek kapasiteye sahip olamaz ve temel ilişkileri bile kaçırır.

![[overfitting.svg]]

### Düzenlileştirme

Modelin eğitim verilerine aşırı uyum sağlamasını önlemek ve genelleme yeteneğini artırmak için uygulanan bir tekniktir. Aşağıda bu tekniğe dair yöntemler verilmiştir.

### Nöron Düşürme

Eğitim sırasında, rastgele bazı aktivasyon fonksiyonları sıfır kabul edilir. Ağın herhangi bir algılayıcıya bel bağlamaması için yapılır. Örneğin, eğitim sırasında katmandaki algılayıcıların yarısı iptal edilebilir.

```python
# TensorFlow
tf.keras.layers.Droput(p=0.5)

# PyTorch
tourch.nn.Dropout(p=0.5)
```

### Erken Durdurma

Modelin eğitim verisini tamamen ezberleyip aşırı uyuma gitmesini önlemek için, maliyetin sıfırlanmasını beklemeden eğitimi erken bir aşamada durdurmaktır.

![[early-stopping.svg]]

Diğer blog yazılarında görüşmek üzere!

### Referanslar

[MIT Intro to Deep Learning](https://introtodeeplearning.com/)