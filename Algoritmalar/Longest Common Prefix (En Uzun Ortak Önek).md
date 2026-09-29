Verilen bir string dizisindeki (kelime listesindeki) tüm kelimelerin ortak olan en uzun başlangıç ekini bulma problemidir.

**Çözüm Mantığı:**
- Dizideki ilk eleman referans alınır ve bu elemanın harfleri üzerinden sırayla bir döngü kurulur.
- Her döngü adımında, o anki harf dizinin geri kalan elemanları ile kıyaslanır.
- Aynı indekslerde bir harf uyuşmazlığı olursa veya harf indeksi kıyaslanan kelimenin boyutunu aşarsa, işlem hemen sonlandırılıp o ana kadar biriken mevcut önek döndürülür.

### TypeScript
```
function longestCommonPrefix(strs: string[]): string { 
	let commonPrefix = ''; 
	let i = 0; 
	let hasMatch = true; 
	while (hasMatch) { 
		let current = strs[0][i]; 
		if(!current) return commonPrefix; 
		for(let j=1; j<strs.length; j++) { 
			if(current !== strs[j][i]) return commonPrefix; 
		} 
		commonPrefix += current; i++; 
	} 
	return commonPrefix; 
}
```

TypeScript'te fonksiyonların dönüş tipi parantezden sonra `:` kullanılarak belirtilebilir. Fonksiyon parametrelerinde ise tip belirtmek iyi bir pratik ve genellikle zorunludur. Değişken tanımlarken `let` veya `const` kullanılır; istenirse `:` ile açıkça tip (Type Annotation) eklenebilir. Tip belirtilmediğinde TypeScript **Type Inference (Tür Çıkarımı)** sayesinde değişkenin türünü otomatik olarak kendisi tahmin eder.

### JavaScript
```
/** 
* @param {string[]} strs 
* @return {string} 
*/ 
var longestCommonPrefix = function(strs) { 
	let commonPrefix = ''; 
	let i = 0; 
	let hasMatch = true; 
	while (hasMatch) { 
		let current = strs[0][i]; 
		if(!current) return commonPrefix; 
		for(let j=1; j<strs.length; j++) { 
			if(current !== strs[j][i]) return commonPrefix; 
		} 
		commonPrefix += current; 
		i++; 
	} 
	return commonPrefix; 
};
```

Bu kod, JavaScript ile yazılmış güzel bir en uzun ortak önek çözümü. JavaScript ile TypeScript sözdizimi temel olarak çok benzer; hatta her iki dilde de fonksiyonlar değişkenlere atanabilir. Fark olarak, JavaScript'te saf TypeScript tipleri yerine yukarıdaki gibi **JSDoc** (`@param`, `@return`) kullanarak veya hiç tip belirtmeden (dinamik olarak) ilerleyebiliriz.

### Python
```
class Solution: 
	def longestCommonPrefix(self, strs: list[str]) -> str: 
		commonPrefix = "" 
		for i in range(len(strs[0])): 
			current = strs[0][i] 
			if current is None: 
				return commonPrefix 
			for j in range(1, len(strs)): 
				if i >= len(strs[j]) or current != strs[j][i]: 
					return commonPrefix 
			commonPrefix += current 
		return commonPrefix
```

Python'da girintilere (indentation) ve blok yapılarına çok dikkat etmek gerekir (`{}` süslü parantezler yerine boşluklar kullanılır). `for` döngüleri genellikle `for i in range(...)` şeklinde yazılır. Python'da tip zorunlu olmasa da fonksiyon imzasında `->` ile dönüş tipi, parametrelerde ise `:` ile veri tipi belirtilebilir (Type Hinting). String ve liste boyutları ise `len()` fonksiyonuyla ölçülür.

### C# 
```
public class Solution { 
	public string LongestCommonPrefix(string[] strs) { 
		string commonPrefix = ""; 
		if (string.IsNullOrEmpty(strs[0])) return commonPrefix; 
		if (strs.Length == 1) return strs[0]; 
		for(int i = 0; i < strs[0].Length; i++) { 
			char current = strs[0][i]; 
			if(char.IsWhiteSpace(current)) break; 
			for(int j = 1; j < strs.Length; j++) { 
				if(i >= strs[j].Length || current != strs[j][i]) 
					return commonPrefix; 
			} 
			commonPrefix += current; 
		} 
		return commonPrefix; 
	} 
}
```

Çıktı tip imzası fonksiyonun başında yazılıyor.
### PHP
```
class Solution { 
	/** 
	* @param String[] $strs 
	* @return String 
	*/ 
	function longestCommonPrefix($strs) { 
		$commonPrefix = ""; 
		$firstItem = $strs[0]; 
		$firstItemLength = strlen($strs[0]); 
		for($i=0; $i<$firstItemLength; $i++) {
			$current = $firstItem[$i]; 
			for($j=1; $j<count($strs); $j++) { 
				if($i >= strlen($strs[$j]) || $current != $strs[$j][$i]) { 
					return $commonPrefix; 
				} 
			} 
			$commonPrefix .= $current; 
		} 
		return $commonPrefix; 
	} 
}
```

Her değişken `$` işareti ile başlıyor. Tip belirtmiyoruz yorum satırı dışında. Dizi boyutu için `count()` , string boyutu için `strlen()` kullanılıyor. İki string'in toplanması `+` yerine `.` ile sağlanıyor.

### Go (Golang)
```
func longestCommonPrefix(strs []string) string { 
	commonPrefix := ""; 
	firstItem := strs[0]; 
	firstItemLength := len(strs[0]); 
	for i := 0; i < firstItemLength; i++ {
		current := firstItem[i]; 
		for j := 1; j < len(strs); j++ { 
			if i >= len(strs[j]) || current != strs[j][i] { 
				return commonPrefix; 
			} 
		} 
		commonPrefix += string(current); 
	} 
	return commonPrefix; 
}
```

**Fonksiyon Tanımı:** `func` anahtar kelimesiyle tanımlanır, fonksiyonun dönüş tipi parantezlerin en sonunda belirtilir.

**Değişken Tanımlama:** Matematiksel tanımlamaya benzer şekilde `:=` operatörü ile kısa tip çıkarımı (short variable declaration) yapılarak tanımlanır.

**Veri Tipleri:** Dizi/Dilim (Slice) tanımları `[]string` şeklinde köşeli parantezle yapılır.

**Sözdizimi Kuralları (Kritik):** `for` ve `if` koşullarında parantez `()` kullanmak zorunlu değildir. Ancak açılış süslü parantezi `{` **kesinlikle aynı satırda başlamalıdır**; aksi takdirde Go derleyicisi otomatik noktalı virgül (ASI) kuralı yüzünden hata verir.

**Boyut ve Karakterler:** Boyutlar `len()` ile ölçülür. String indeksine erişildiğinde `byte` (`uint8`) türünde değer döner; bu yüzden toplama işleminde kullanmak için `string(current)` şeklinde cast (dönüşüm) yapmak gerekir.