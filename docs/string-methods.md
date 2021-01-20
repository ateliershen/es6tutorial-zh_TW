# 字串的新增方法

本章介紹字串物件的新增方法。

## String.fromCodePoint()

ES5 提供`String.fromCharCode()`方法，用於從 Unicode 碼點返回對應字元，但是這個方法不能識別碼點大於`0xFFFF`的字元。

```javascript
String.fromCharCode(0x20BB7)
// "ஷ"
```

上面程式碼中，`String.fromCharCode()`不能識別大於`0xFFFF`的碼點，所以`0x20BB7`就發生了溢位，最高位`2`被捨棄了，最後返回碼點`U+0BB7`對應的字元，而不是碼點`U+20BB7`對應的字元。

ES6 提供了`String.fromCodePoint()`方法，可以識別大於`0xFFFF`的字元，彌補了`String.fromCharCode()`方法的不足。在作用上，正好與下面的`codePointAt()`方法相反。

```javascript
String.fromCodePoint(0x20BB7)
// "𠮷"
String.fromCodePoint(0x78, 0x1f680, 0x79) === 'x\uD83D\uDE80y'
// true
```

上面程式碼中，如果`String.fromCodePoint`方法有多個引數，則它們會被合併成一個字串返回。

注意，`fromCodePoint`方法定義在`String`物件上，而`codePointAt`方法定義在字串的例項物件上。

## String.raw()

ES6 還為原生的 String 物件，提供了一個`raw()`方法。該方法返回一個斜槓都被轉義（即斜槓前面再加一個斜槓）的字串，往往用於模板字串的處理方法。

```javascript
String.raw`Hi\n${2+3}!`
// 實際返回 "Hi\\n5!"，顯示的是轉義後的結果 "Hi\n5!"

String.raw`Hi\u000A!`;
// 實際返回 "Hi\\u000A!"，顯示的是轉義後的結果 "Hi\u000A!"
```

如果原字串的斜槓已經轉義，那麼`String.raw()`會進行再次轉義。

```javascript
String.raw`Hi\\n`
// 返回 "Hi\\\\n"

String.raw`Hi\\n` === "Hi\\\\n" // true
```

`String.raw()`方法可以作為處理模板字串的基本方法，它會將所有變數替換，而且對斜槓進行轉義，方便下一步作為字串來使用。

`String.raw()`本質上是一個正常的函式，只是專用於模板字串的標籤函式。如果寫成正常函式的形式，它的第一個引數，應該是一個具有`raw`屬性的物件，且`raw`屬性的值應該是一個數組，對應模板字串解析後的值。

```javascript
// `foo${1 + 2}bar`
// 等同於
String.raw({ raw: ['foo', 'bar'] }, 1 + 2) // "foo3bar"
```

上面程式碼中，`String.raw()`方法的第一個引數是一個物件，它的`raw`屬性等同於原始的模板字串解析後得到的陣列。

作為函式，`String.raw()`的程式碼實現基本如下。

```javascript
String.raw = function (strings, ...values) {
  let output = '';
  let index;
  for (index = 0; index < values.length; index++) {
    output += strings.raw[index] + values[index];
  }

  output += strings.raw[index]
  return output;
}
```

## 例項方法：codePointAt()

JavaScript 內部，字元以 UTF-16 的格式儲存，每個字元固定為`2`個位元組。對於那些需要`4`個位元組儲存的字元（Unicode 碼點大於`0xFFFF`的字元），JavaScript 會認為它們是兩個字元。

```javascript
var s = "𠮷";

s.length // 2
s.charAt(0) // ''
s.charAt(1) // ''
s.charCodeAt(0) // 55362
s.charCodeAt(1) // 57271
```

上面程式碼中，漢字“𠮷”（注意，這個字不是“吉祥”的“吉”）的碼點是`0x20BB7`，UTF-16 編碼為`0xD842 0xDFB7`（十進位制為`55362 57271`），需要`4`個位元組儲存。對於這種`4`個位元組的字元，JavaScript 不能正確處理，字串長度會誤判為`2`，而且`charAt()`方法無法讀取整個字元，`charCodeAt()`方法只能分別返回前兩個位元組和後兩個位元組的值。

ES6 提供了`codePointAt()`方法，能夠正確處理 4 個位元組儲存的字元，返回一個字元的碼點。

```javascript
let s = '𠮷a';

s.codePointAt(0) // 134071
s.codePointAt(1) // 57271

s.codePointAt(2) // 97
```

`codePointAt()`方法的引數，是字元在字串中的位置（從 0 開始）。上面程式碼中，JavaScript 將“𠮷a”視為三個字元，codePointAt 方法在第一個字元上，正確地識別了“𠮷”，返回了它的十進位制碼點 134071（即十六進位制的`20BB7`）。在第二個字元（即“𠮷”的後兩個位元組）和第三個字元“a”上，`codePointAt()`方法的結果與`charCodeAt()`方法相同。

總之，`codePointAt()`方法會正確返回 32 位的 UTF-16 字元的碼點。對於那些兩個位元組儲存的常規字元，它的返回結果與`charCodeAt()`方法相同。

`codePointAt()`方法返回的是碼點的十進位制值，如果想要十六進位制的值，可以使用`toString()`方法轉換一下。

```javascript
let s = '𠮷a';

s.codePointAt(0).toString(16) // "20bb7"
s.codePointAt(2).toString(16) // "61"
```

你可能注意到了，`codePointAt()`方法的引數，仍然是不正確的。比如，上面程式碼中，字元`a`在字串`s`的正確位置序號應該是 1，但是必須向`codePointAt()`方法傳入 2。解決這個問題的一個辦法是使用`for...of`迴圈，因為它會正確識別 32 位的 UTF-16 字元。

```javascript
let s = '𠮷a';
for (let ch of s) {
  console.log(ch.codePointAt(0).toString(16));
}
// 20bb7
// 61
```

另一種方法也可以，使用擴充套件運算子（`...`）進行展開運算。

```javascript
let arr = [...'𠮷a']; // arr.length === 2
arr.forEach(
  ch => console.log(ch.codePointAt(0).toString(16))
);
// 20bb7
// 61
```

`codePointAt()`方法是測試一個字元由兩個位元組還是由四個位元組組成的最簡單方法。

```javascript
function is32Bit(c) {
  return c.codePointAt(0) > 0xFFFF;
}

is32Bit("𠮷") // true
is32Bit("a") // false
```

## 例項方法：normalize()

許多歐洲語言有語調符號和重音符號。為了表示它們，Unicode 提供了兩種方法。一種是直接提供帶重音符號的字元，比如`Ǒ`（\u01D1）。另一種是提供合成符號（combining character），即原字元與重音符號的合成，兩個字符合成一個字元，比如`O`（\u004F）和`ˇ`（\u030C）合成`Ǒ`（\u004F\u030C）。

這兩種表示方法，在視覺和語義上都等價，但是 JavaScript 不能識別。

```javascript
'\u01D1'==='\u004F\u030C' //false

'\u01D1'.length // 1
'\u004F\u030C'.length // 2
```

上面程式碼表示，JavaScript 將合成字元視為兩個字元，導致兩種表示方法不相等。

ES6 提供字串例項的`normalize()`方法，用來將字元的不同表示方法統一為同樣的形式，這稱為 Unicode 正規化。

```javascript
'\u01D1'.normalize() === '\u004F\u030C'.normalize()
// true
```

`normalize`方法可以接受一個引數來指定`normalize`的方式，引數的四個可選值如下。

- `NFC`，預設引數，表示“標準等價合成”（Normalization Form Canonical Composition），返回多個簡單字元的合成字元。所謂“標準等價”指的是視覺和語義上的等價。
- `NFD`，表示“標準等價分解”（Normalization Form Canonical Decomposition），即在標準等價的前提下，返回合成字元分解的多個簡單字元。
- `NFKC`，表示“相容等價合成”（Normalization Form Compatibility Composition），返回合成字元。所謂“相容等價”指的是語義上存在等價，但視覺上不等價，比如“囍”和“喜喜”。（這只是用來舉例，`normalize`方法不能識別中文。）
- `NFKD`，表示“相容等價分解”（Normalization Form Compatibility Decomposition），即在相容等價的前提下，返回合成字元分解的多個簡單字元。

```javascript
'\u004F\u030C'.normalize('NFC').length // 1
'\u004F\u030C'.normalize('NFD').length // 2
```

上面程式碼表示，`NFC`引數返回字元的合成形式，`NFD`引數返回字元的分解形式。

不過，`normalize`方法目前不能識別三個或三個以上字元的合成。這種情況下，還是隻能使用正則表示式，透過 Unicode 編號區間判斷。

## 例項方法：includes(), startsWith(), endsWith()

傳統上，JavaScript 只有`indexOf`方法，可以用來確定一個字串是否包含在另一個字串中。ES6 又提供了三種新方法。

- **includes()**：返回布林值，表示是否找到了引數字串。
- **startsWith()**：返回布林值，表示引數字串是否在原字串的頭部。
- **endsWith()**：返回布林值，表示引數字串是否在原字串的尾部。

```javascript
let s = 'Hello world!';

s.startsWith('Hello') // true
s.endsWith('!') // true
s.includes('o') // true
```

這三個方法都支援第二個引數，表示開始搜尋的位置。

```javascript
let s = 'Hello world!';

s.startsWith('world', 6) // true
s.endsWith('Hello', 5) // true
s.includes('Hello', 6) // false
```

上面程式碼表示，使用第二個引數`n`時，`endsWith`的行為與其他兩個方法有所不同。它針對前`n`個字元，而其他兩個方法針對從第`n`個位置直到字串結束。

## 例項方法：repeat()

`repeat`方法返回一個新字串，表示將原字串重複`n`次。

```javascript
'x'.repeat(3) // "xxx"
'hello'.repeat(2) // "hellohello"
'na'.repeat(0) // ""
```

引數如果是小數，會被取整。

```javascript
'na'.repeat(2.9) // "nana"
```

如果`repeat`的引數是負數或者`Infinity`，會報錯。

```javascript
'na'.repeat(Infinity)
// RangeError
'na'.repeat(-1)
// RangeError
```

但是，如果引數是 0 到-1 之間的小數，則等同於 0，這是因為會先進行取整運算。0 到-1 之間的小數，取整以後等於`-0`，`repeat`視同為 0。

```javascript
'na'.repeat(-0.9) // ""
```

引數`NaN`等同於 0。

```javascript
'na'.repeat(NaN) // ""
```

如果`repeat`的引數是字串，則會先轉換成數字。

```javascript
'na'.repeat('na') // ""
'na'.repeat('3') // "nanana"
```

## 例項方法：padStart()，padEnd()

ES2017 引入了字串補全長度的功能。如果某個字串不夠指定長度，會在頭部或尾部補全。`padStart()`用於頭部補全，`padEnd()`用於尾部補全。

```javascript
'x'.padStart(5, 'ab') // 'ababx'
'x'.padStart(4, 'ab') // 'abax'

'x'.padEnd(5, 'ab') // 'xabab'
'x'.padEnd(4, 'ab') // 'xaba'
```

上面程式碼中，`padStart()`和`padEnd()`一共接受兩個引數，第一個引數是字串補全生效的最大長度，第二個引數是用來補全的字串。

如果原字串的長度，等於或大於最大長度，則字串補全不生效，返回原字串。

```javascript
'xxx'.padStart(2, 'ab') // 'xxx'
'xxx'.padEnd(2, 'ab') // 'xxx'
```

如果用來補全的字串與原字串，兩者的長度之和超過了最大長度，則會截去超出位數的補全字串。

```javascript
'abc'.padStart(10, '0123456789')
// '0123456abc'
```

如果省略第二個引數，預設使用空格補全長度。

```javascript
'x'.padStart(4) // '   x'
'x'.padEnd(4) // 'x   '
```

`padStart()`的常見用途是為數值補全指定位數。下面程式碼生成 10 位的數值字串。

```javascript
'1'.padStart(10, '0') // "0000000001"
'12'.padStart(10, '0') // "0000000012"
'123456'.padStart(10, '0') // "0000123456"
```

另一個用途是提示字串格式。

```javascript
'12'.padStart(10, 'YYYY-MM-DD') // "YYYY-MM-12"
'09-12'.padStart(10, 'YYYY-MM-DD') // "YYYY-09-12"
```

## 例項方法：trimStart()，trimEnd()

[ES2019](https://github.com/tc39/proposal-string-left-right-trim) 對字串例項新增了`trimStart()`和`trimEnd()`這兩個方法。它們的行為與`trim()`一致，`trimStart()`消除字串頭部的空格，`trimEnd()`消除尾部的空格。它們返回的都是新字串，不會修改原始字串。

```javascript
const s = '  abc  ';

s.trim() // "abc"
s.trimStart() // "abc  "
s.trimEnd() // "  abc"
```

上面程式碼中，`trimStart()`只消除頭部的空格，保留尾部的空格。`trimEnd()`也是類似行為。

除了空格鍵，這兩個方法對字串頭部（或尾部）的 tab 鍵、換行符等不可見的空白符號也有效。

瀏覽器還部署了額外的兩個方法，`trimLeft()`是`trimStart()`的別名，`trimRight()`是`trimEnd()`的別名。

## 例項方法：matchAll()

`matchAll()`方法返回一個正則表示式在當前字串的所有匹配，詳見《正則的擴充套件》的一章。

## 例項方法：replaceAll()

歷史上，字串的例項方法`replace()`只能替換第一個匹配。

```javascript
'aabbcc'.replace('b', '_')
// 'aa_bcc'
```

上面例子中，`replace()`只將第一個`b`替換成了下劃線。

如果要替換所有的匹配，不得不使用正則表示式的`g`修飾符。

```javascript
'aabbcc'.replace(/b/g, '_')
// 'aa__cc'
```

正則表示式畢竟不是那麼方便和直觀，[ES2021](https://github.com/tc39/proposal-string-replaceall) 引入了`replaceAll()`方法，可以一次性替換所有匹配。

```javascript
'aabbcc'.replaceAll('b', '_')
// 'aa__cc'
```

它的用法與`replace()`相同，返回一個新字串，不會改變原字串。

```javascript
String.prototype.replaceAll(searchValue, replacement)
```

上面程式碼中，`searchValue`是搜尋模式，可以是一個字串，也可以是一個全域性的正則表示式（帶有`g`修飾符）。

如果`searchValue`是一個不帶有`g`修飾符的正則表示式，`replaceAll()`會報錯。這一點跟`replace()`不同。

```javascript
// 不報錯
'aabbcc'.replace(/b/, '_')

// 報錯
'aabbcc'.replaceAll(/b/, '_')
```

上面例子中，`/b/`不帶有`g`修飾符，會導致`replaceAll()`報錯。

`replaceAll()`的第二個引數`replacement`是一個字串，表示替換的文字，其中可以使用一些特殊字串。

- `$&`：匹配的子字串。
- `` $` ``：匹配結果前面的文字。
- `$'`：匹配結果後面的文字。
- `$n`：匹配成功的第`n`組內容，`n`是從1開始的自然數。這個引數生效的前提是，第一個引數必須是正則表示式。
- `$$`：指代美元符號`$`。

下面是一些例子。

```javascript
// $& 表示匹配的字串，即`b`本身
// 所以返回結果與原字串一致
'abbc'.replaceAll('b', '$&')
// 'abbc'

// $` 表示匹配結果之前的字串
// 對於第一個`b`，$` 指代`a`
// 對於第二個`b`，$` 指代`ab`
'abbc'.replaceAll('b', '$`')
// 'aaabc'

// $' 表示匹配結果之後的字串
// 對於第一個`b`，$' 指代`bc`
// 對於第二個`b`，$' 指代`c`
'abbc'.replaceAll('b', `$'`)
// 'abccc'

// $1 表示正則表示式的第一個組匹配，指代`ab`
// $2 表示正則表示式的第二個組匹配，指代`bc`
'abbc'.replaceAll(/(ab)(bc)/g, '$2$1')
// 'bcab'

// $$ 指代 $
'abc'.replaceAll('b', '$$')
// 'a$c'
```

`replaceAll()`的第二個引數`replacement`除了為字串，也可以是一個函式，該函式的返回值將替換掉第一個引數`searchValue`匹配的文字。

```javascript
'aabbcc'.replaceAll('b', () => '_')
// 'aa__cc'
```

上面例子中，`replaceAll()`的第二個引數是一個函式，該函式的返回值會替換掉所有`b`的匹配。

這個替換函式可以接受多個引數。第一個引數是捕捉到的匹配內容，第二個引數捕捉到是組匹配（有多少個組匹配，就有多少個對應的引數）。此外，最後還可以新增兩個引數，倒數第二個引數是捕捉到的內容在整個字串中的位置，最後一個引數是原字串。

```javascript
const str = '123abc456';
const regex = /(\d+)([a-z]+)(\d+)/g;

function replacer(match, p1, p2, p3, offset, string) {
  return [p1, p2, p3].join(' - ');
}

str.replaceAll(regex, replacer)
// 123 - abc - 456
```

上面例子中，正則表示式有三個組匹配，所以`replacer()`函式的第一個引數`match`是捕捉到的匹配內容（即字串`123abc456`），後面三個引數`p1`、`p2`、`p3`則依次為三個組匹配。

