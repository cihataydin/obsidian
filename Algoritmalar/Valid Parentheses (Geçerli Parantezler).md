### Problem

Açılan her parantez aynı çeşit parantez ve aynı sıra ile kapatılmalıdır. Kapatılan her parantez doğru açılış paranteziyle açılmış olmalıdır.
Örneğin,

**Girdi:** `s = "()[]{}"`, **Çıktı:** `true`
**Girdi:** `s = "([)]"` **Çıktı:** `false`
**Girdi:** `s = "([])"` **Çıktı:** `true`

Çözüm için `stack` yapısını kullanacağız ve verilen `string` de `char` lar üzerine döngü kuracağız. ==LIFO== mantığını kullanacağız. Her kapalı parantez için `stack` e eklenen son elemanı kontrol edip eşleşmiş ise `stack` den sileceğiz yoksa `stack` e ekleyeceğiz. Böylece nested yapıda dahi içerden dışarıya doğru parantezleri kontrol etmiş olacağız.

### TypeScirpt

Dizi tanımlayıp direk `stack` miş gibi işlem yapabildim. `const .. of ..` yapısını öğrendim. `Object` yerine `Record` kullanabilceğimi hatırladım. $(4ms)$

```typescript
function isValid(s: string): boolean { 
	const stack: Array<string> = []; 
	const chars: Record<string, string> = { ')': '(', '}': '{', ']': '[' } 
	for (const c of s) { 
		if(chars[c]) { 
			if(stack[stack.length -1] === chars[c]) { 
				stack.pop(); 
			} 
			else { 
				stack.push(c); 
			} 
		} 
		else {
			stack.push(c); 
		} 
	} 
	return stack.length === 0; 
};
```

### JavaScript

`TypeScript` kodundaki tip tanımlarını silip direk aynı çözümü kullandım. $(4ms)$

```javascript
/** 
* @param {string} s 
* @return {boolean} 
*/ 
var isValid = function(s) { 
	const stack = []; 
	const chars = { ')': '(', '}': '{', ']': '[' } 
	for (const c of s) { 
		if(chars[c]) { 
			if(stack[stack.length -1] === chars[c]) { 
				stack.pop(); 
			} 
			else{ 
				stack.push(c); 
			} 
		} 
		else { 
			stack.push(c); 
		} 
	} 
	return stack.length === 0; 
};
```

### C\#

Sözlüğün `static` ve `readonly` olmasına dikkatinizi çekerim. `C#` ın kendinde tanımlı olan `Stack` tipini kullandım. $(6ms)$

```csharp
public class Solution { 
	static readonly Dictionary<char, char> chars = new Dictionary<char, char>{ 
		{ ')', '(' }, 
		{ '}', '{' }, 
		{ ']', '[' } 
	}; 
	public bool IsValid(string s) { 
		Stack<char> stack = []; 
		for(int i=0; i<s.Length; i++) { 
			char c = s[i]; 
			if(chars.ContainsKey(c)) { 
				if( stack.Count > 0 && stack.Peek() == chars[c]) { 
					stack.Pop(); 
				} 
				else { 
					stack.Push(c); 
				} 
			} 
			else { 
				stack.Push(c); 
			} 
		} 
		return stack.Count == 0; 
	} 
}
```

### Python

`liste` tipine `stack` miş gibi davranabildim. `Python` da bir listenin boş olduğunu `not` keyword'ü ile kontrol edebildim. `liste` sonundaki elemana $-1$ indeks ile ulaşabildim. $(<1ms)$

```python
class Solution:Stack 
	def isValid(self, s: str) -> bool: 
		""" :type s: str 
			:rtype: bool 
		""" 
		stack = [] 
		chars = { ')': '(', '}': '{', ']': '[' } 
		for c in s: 
			if c in chars: 
				if stack and stack[-1] == chars[c]: 
					stack.pop() 
				else: 
					stack.append(c) 
			else: 
				stack.append(c) 
		return not stack
```

### PHP

Her değişkenin başına `$` koymayı unutuyorum. Dizi tanımlayıp direk `stack` miş gibi davranabildim. $(1ms)$

```php
class Solution { 
	/** 
	* @param String $s 
	* @return Boolean 
	*/ 
	function isValid($s) { 
		$stack = []; 
		$chars = [ ')' => '(', '}' => '{', ']' => '[' ]; 
		for($i = 0; $i < strlen($s); $i++) { 
			$c = $s[$i]; 
			if(isset($chars[$c])) { 
				if($stack[count($stack) - 1] == $chars[$c]) { 
					array_pop($stack); 
				} 
				else { 
					array_push($stack, $c); 
				} 
			} 
			else { 
				array_push($stack, $c); 
			} 
		} 
		return empty($stack); 
	} 
}
```

### Go

`} else {` şeklinde yazılması zorunludur. `rune` tipi sembol anlamına gelip bir karakterin birden fazla `byte` a karşılık gelebileceğini vurgulamak için tanımlanmış. Geleneksel `char` $1$ `byte` ($8$ `bit`) anlayışını yıkmışlar. $1$ `rune` $4$ `byte` dır. Ayrıca `for` döngüsünde `range` kullandım. $(<1ms)$

```go
func isValid(s string) bool { 
	chars := map[rune]rune{ ')': '(', '}': '{', ']': '[', } 
	var stack []rune 
	for _, c := range s { 
		if chars[c] != 0 { 
			if len(stack) != 0 && stack[len(stack)-1] == chars[c] { 
				stack = stack[:len(stack)-1] 
			} else { 
				stack = append(stack, c) 
			} 
		} else { 
			stack = append(stack, c) 
		} 
	} 
	return len(stack) == 0 
}
```

### Sonuç

Algoritma çalışma sürelerinde, $1ms$ den fazla süren diller için optimizasyona gidilebilir, static sözlük yerine `switch` kullanarak. Lakin kod okunabilirliğine de önem veriyorum. Bu problem özelinde favori dil olarak `go` seçerim.

Diğer blog yazılarında görüşmek üzere!