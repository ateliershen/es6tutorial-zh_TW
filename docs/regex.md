# 正則的擴充套件

## RegExp 建構函式

在 ES5 中，`RegExp`建構函式的引數有兩種情況。

第一種情況是，引數是字串，這時第二個引數表示正則表示式的修飾符（flag）。

```javascript
var regex = new RegExp('xyz', 'i');
// 等價於
var regex = /xyz/i;
```

第二種情況是，引數是一個正則表示式，這時會返回一個原有正則表示式的複製。

```javascript
var regex = new RegExp(/xyz/i);
// 等價於
var regex = /xyz/i;
```

但是，ES5 不允許此時使用第二個引數新增修飾符，否則會報錯。

```javascript
var regex = new RegExp(/xyz/, 'i');
// Uncaught TypeError: Cannot supply flags when constructing one RegExp from another
```

ES6 改變了這種行為。如果`RegExp`建構函式第一個引數是一個正則物件，那麼可以使用第二個引數指定修飾符。而且，返回的正則表示式會忽略原有的正則表示式的修飾符，只使用新指定的修飾符。

```javascript
new RegExp(/abc/ig, 'i').flags
// "i"
```

上面程式碼中，原有正則物件的修飾符是`ig`，它會被第二個引數`i`覆蓋。

## 字串的正則方法

ES6 出現之前，字串物件共有 4 個方法，可以使用正則表示式：`match()`、`replace()`、`search()`和`split()`。

ES6 將這 4 個方法，在語言內部全部呼叫`RegExp`的例項方法，從而做到所有與正則相關的方法，全都定義在`RegExp`物件上。

- `String.prototype.match` 呼叫 `RegExp.prototype[Symbol.match]`
- `String.prototype.replace` 呼叫 `RegExp.prototype[Symbol.replace]`
- `String.prototype.search` 呼叫 `RegExp.prototype[Symbol.search]`
- `String.prototype.split` 呼叫 `RegExp.prototype[Symbol.split]`

## u 修飾符

ES6 對正則表示式添加了`u`修飾符，含義為“Unicode 模式”，用來正確處理大於`\uFFFF`的 Unicode 字元。也就是說，會正確處理四個位元組的 UTF-16 編碼。

```javascript
/^\uD83D/u.test('\uD83D\uDC2A') // false
/^\uD83D/.test('\uD83D\uDC2A') // true
```

上面程式碼中，`\uD83D\uDC2A`是一個四個位元組的 UTF-16 編碼，代表一個字元。但是，ES5 不支援四個位元組的 UTF-16 編碼，會將其識別為兩個字元，導致第二行程式碼結果為`true`。加了`u`修飾符以後，ES6 就會識別其為一個字元，所以第一行程式碼結果為`false`。

一旦加上`u`修飾符號，就會修改下面這些正則表示式的行為。

**（1）點字元**

點（`.`）字元在正則表示式中，含義是除了換行符以外的任意單個字元。對於碼點大於`0xFFFF`的 Unicode 字元，點字元不能識別，必須加上`u`修飾符。

```javascript
var s = '𠮷';

/^.$/.test(s) // false
/^.$/u.test(s) // true
```

上面程式碼表示，如果不新增`u`修飾符，正則表示式就會認為字串為兩個字元，從而匹配失敗。

**（2）Unicode 字元表示法**

ES6 新增了使用大括號表示 Unicode 字元，這種表示法在正則表示式中必須加上`u`修飾符，才能識別當中的大括號，否則會被解讀為量詞。

```javascript
/\u{61}/.test('a') // false
/\u{61}/u.test('a') // true
/\u{20BB7}/u.test('𠮷') // true
```

上面程式碼表示，如果不加`u`修飾符，正則表示式無法識別`\u{61}`這種表示法，只會認為這匹配 61 個連續的`u`。

**（3）量詞**

使用`u`修飾符後，所有量詞都會正確識別碼點大於`0xFFFF`的 Unicode 字元。

```javascript
/a{2}/.test('aa') // true
/a{2}/u.test('aa') // true
/𠮷{2}/.test('𠮷𠮷') // false
/𠮷{2}/u.test('𠮷𠮷') // true
```

**（4）預定義模式**

`u`修飾符也影響到預定義模式，能否正確識別碼點大於`0xFFFF`的 Unicode 字元。

```javascript
/^\S$/.test('𠮷') // false
/^\S$/u.test('𠮷') // true
```

上面程式碼的`\S`是預定義模式，匹配所有非空白字元。只有加了`u`修飾符，它才能正確匹配碼點大於`0xFFFF`的 Unicode 字元。

利用這一點，可以寫出一個正確返回字串長度的函式。

```javascript
function codePointLength(text) {
  var result = text.match(/[\s\S]/gu);
  return result ? result.length : 0;
}

var s = '𠮷𠮷';

s.length // 4
codePointLength(s) // 2
```

**（5）i 修飾符**

有些 Unicode 字元的編碼不同，但是字型很相近，比如，`\u004B`與`\u212A`都是大寫的`K`。

```javascript
/[a-z]/i.test('\u212A') // false
/[a-z]/iu.test('\u212A') // true
```

上面程式碼中，不加`u`修飾符，就無法識別非規範的`K`字元。

**（6）轉義**

沒有`u`修飾符的情況下，正則中沒有定義的轉義（如逗號的轉義`\,`）無效，而在`u`模式會報錯。

```javascript
/\,/ // /\,/
/\,/u // 報錯
```

上面程式碼中，沒有`u`修飾符時，逗號前面的反斜槓是無效的，加了`u`修飾符就報錯。

## RegExp.prototype.unicode 屬性

正則例項物件新增`unicode`屬性，表示是否設定了`u`修飾符。

```javascript
const r1 = /hello/;
const r2 = /hello/u;

r1.unicode // false
r2.unicode // true
```

上面程式碼中，正則表示式是否設定了`u`修飾符，可以從`unicode`屬性看出來。

## y 修飾符

除了`u`修飾符，ES6 還為正則表示式添加了`y`修飾符，叫做“粘連”（sticky）修飾符。

`y`修飾符的作用與`g`修飾符類似，也是全域性匹配，後一次匹配都從上一次匹配成功的下一個位置開始。不同之處在於，`g`修飾符只要剩餘位置中存在匹配就可，而`y`修飾符確保匹配必須從剩餘的第一個位置開始，這也就是“粘連”的涵義。

```javascript
var s = 'aaa_aa_a';
var r1 = /a+/g;
var r2 = /a+/y;

r1.exec(s) // ["aaa"]
r2.exec(s) // ["aaa"]

r1.exec(s) // ["aa"]
r2.exec(s) // null
```

上面程式碼有兩個正則表示式，一個使用`g`修飾符，另一個使用`y`修飾符。這兩個正則表示式各執行了兩次，第一次執行的時候，兩者行為相同，剩餘字串都是`_aa_a`。由於`g`修飾沒有位置要求，所以第二次執行會返回結果，而`y`修飾符要求匹配必須從頭部開始，所以返回`null`。

如果改一下正則表示式，保證每次都能頭部匹配，`y`修飾符就會返回結果了。

```javascript
var s = 'aaa_aa_a';
var r = /a+_/y;

r.exec(s) // ["aaa_"]
r.exec(s) // ["aa_"]
```

上面程式碼每次匹配，都是從剩餘字串的頭部開始。

使用`lastIndex`屬性，可以更好地說明`y`修飾符。

```javascript
const REGEX = /a/g;

// 指定從2號位置（y）開始匹配
REGEX.lastIndex = 2;

// 匹配成功
const match = REGEX.exec('xaya');

// 在3號位置匹配成功
match.index // 3

// 下一次匹配從4號位開始
REGEX.lastIndex // 4

// 4號位開始匹配失敗
REGEX.exec('xaya') // null
```

上面程式碼中，`lastIndex`屬性指定每次搜尋的開始位置，`g`修飾符從這個位置開始向後搜尋，直到發現匹配為止。

`y`修飾符同樣遵守`lastIndex`屬性，但是要求必須在`lastIndex`指定的位置發現匹配。

```javascript
const REGEX = /a/y;

// 指定從2號位置開始匹配
REGEX.lastIndex = 2;

// 不是粘連，匹配失敗
REGEX.exec('xaya') // null

// 指定從3號位置開始匹配
REGEX.lastIndex = 3;

// 3號位置是粘連，匹配成功
const match = REGEX.exec('xaya');
match.index // 3
REGEX.lastIndex // 4
```

實際上，`y`修飾符號隱含了頭部匹配的標誌`^`。

```javascript
/b/y.exec('aba')
// null
```

上面程式碼由於不能保證頭部匹配，所以返回`null`。`y`修飾符的設計本意，就是讓頭部匹配的標誌`^`在全域性匹配中都有效。

下面是字串物件的`replace`方法的例子。

```javascript
const REGEX = /a/gy;
'aaxa'.replace(REGEX, '-') // '--xa'
```

上面程式碼中，最後一個`a`因為不是出現在下一次匹配的頭部，所以不會被替換。

單單一個`y`修飾符對`match`方法，只能返回第一個匹配，必須與`g`修飾符聯用，才能返回所有匹配。

```javascript
'a1a2a3'.match(/a\d/y) // ["a1"]
'a1a2a3'.match(/a\d/gy) // ["a1", "a2", "a3"]
```

`y`修飾符的一個應用，是從字串提取 token（詞元），`y`修飾符確保了匹配之間不會有漏掉的字元。

```javascript
const TOKEN_Y = /\s*(\+|[0-9]+)\s*/y;
const TOKEN_G  = /\s*(\+|[0-9]+)\s*/g;

tokenize(TOKEN_Y, '3 + 4')
// [ '3', '+', '4' ]
tokenize(TOKEN_G, '3 + 4')
// [ '3', '+', '4' ]

function tokenize(TOKEN_REGEX, str) {
  let result = [];
  let match;
  while (match = TOKEN_REGEX.exec(str)) {
    result.push(match[1]);
  }
  return result;
}
```

上面程式碼中，如果字串裡面沒有非法字元，`y`修飾符與`g`修飾符的提取結果是一樣的。但是，一旦出現非法字元，兩者的行為就不一樣了。

```javascript
tokenize(TOKEN_Y, '3x + 4')
// [ '3' ]
tokenize(TOKEN_G, '3x + 4')
// [ '3', '+', '4' ]
```

上面程式碼中，`g`修飾符會忽略非法字元，而`y`修飾符不會，這樣就很容易發現錯誤。

## RegExp.prototype.sticky 屬性

與`y`修飾符相匹配，ES6 的正則例項物件多了`sticky`屬性，表示是否設定了`y`修飾符。

```javascript
var r = /hello\d/y;
r.sticky // true
```

## RegExp.prototype.flags 屬性

ES6 為正則表示式新增了`flags`屬性，會返回正則表示式的修飾符。

```javascript
// ES5 的 source 屬性
// 返回正則表示式的正文
/abc/ig.source
// "abc"

// ES6 的 flags 屬性
// 返回正則表示式的修飾符
/abc/ig.flags
// 'gi'
```

## s 修飾符：dotAll 模式

正則表示式中，點（`.`）是一個特殊字元，代表任意的單個字元，但是有兩個例外。一個是四個位元組的 UTF-16 字元，這個可以用`u`修飾符解決；另一個是行終止符（line terminator character）。

所謂行終止符，就是該字元表示一行的終結。以下四個字元屬於“行終止符”。

- U+000A 換行符（`\n`）
- U+000D 回車符（`\r`）
- U+2028 行分隔符（line separator）
- U+2029 段分隔符（paragraph separator）

```javascript
/foo.bar/.test('foo\nbar')
// false
```

上面程式碼中，因為`.`不匹配`\n`，所以正則表示式返回`false`。

但是，很多時候我們希望匹配的是任意單個字元，這時有一種變通的寫法。

```javascript
/foo[^]bar/.test('foo\nbar')
// true
```

這種解決方案畢竟不太符合直覺，ES2018 [引入](https://github.com/tc39/proposal-regexp-dotall-flag)`s`修飾符，使得`.`可以匹配任意單個字元。

```javascript
/foo.bar/s.test('foo\nbar') // true
```

這被稱為`dotAll`模式，即點（dot）代表一切字元。所以，正則表示式還引入了一個`dotAll`屬性，返回一個布林值，表示該正則表示式是否處在`dotAll`模式。

```javascript
const re = /foo.bar/s;
// 另一種寫法
// const re = new RegExp('foo.bar', 's');

re.test('foo\nbar') // true
re.dotAll // true
re.flags // 's'
```

`/s`修飾符和多行修飾符`/m`不衝突，兩者一起使用的情況下，`.`匹配所有字元，而`^`和`$`匹配每一行的行首和行尾。

## 後行斷言

JavaScript 語言的正則表示式，只支援先行斷言（lookahead）和先行否定斷言（negative lookahead），不支援後行斷言（lookbehind）和後行否定斷言（negative lookbehind）。ES2018 引入[後行斷言](https://github.com/tc39/proposal-regexp-lookbehind)，V8 引擎 4.9 版（Chrome 62）已經支援。

“先行斷言”指的是，`x`只有在`y`前面才匹配，必須寫成`/x(?=y)/`。比如，只匹配百分號之前的數字，要寫成`/\d+(?=%)/`。“先行否定斷言”指的是，`x`只有不在`y`前面才匹配，必須寫成`/x(?!y)/`。比如，只匹配不在百分號之前的數字，要寫成`/\d+(?!%)/`。

```javascript
/\d+(?=%)/.exec('100% of US presidents have been male')  // ["100"]
/\d+(?!%)/.exec('that’s all 44 of them')                 // ["44"]
```

上面兩個字串，如果互換正則表示式，就不會得到相同結果。另外，還可以看到，“先行斷言”括號之中的部分（`(?=%)`），是不計入返回結果的。

“後行斷言”正好與“先行斷言”相反，`x`只有在`y`後面才匹配，必須寫成`/(?<=y)x/`。比如，只匹配美元符號之後的數字，要寫成`/(?<=\$)\d+/`。“後行否定斷言”則與“先行否定斷言”相反，`x`只有不在`y`後面才匹配，必須寫成`/(?<!y)x/`。比如，只匹配不在美元符號後面的數字，要寫成`/(?<!\$)\d+/`。

```javascript
/(?<=\$)\d+/.exec('Benjamin Franklin is on the $100 bill')  // ["100"]
/(?<!\$)\d+/.exec('it’s is worth about €90')                // ["90"]
```

上面的例子中，“後行斷言”的括號之中的部分（`(?<=\$)`），也是不計入返回結果。

下面的例子是使用後行斷言進行字串替換。

```javascript
const RE_DOLLAR_PREFIX = /(?<=\$)foo/g;
'$foo %foo foo'.replace(RE_DOLLAR_PREFIX, 'bar');
// '$bar %foo foo'
```

上面程式碼中，只有在美元符號後面的`foo`才會被替換。

“後行斷言”的實現，需要先匹配`/(?<=y)x/`的`x`，然後再回到左邊，匹配`y`的部分。這種“先右後左”的執行順序，與所有其他正則操作相反，導致了一些不符合預期的行為。

首先，後行斷言的組匹配，與正常情況下結果是不一樣的。

```javascript
/(?<=(\d+)(\d+))$/.exec('1053') // ["", "1", "053"]
/^(\d+)(\d+)$/.exec('1053') // ["1053", "105", "3"]
```

上面程式碼中，需要捕捉兩個組匹配。沒有“後行斷言”時，第一個括號是貪婪模式，第二個括號只能捕獲一個字元，所以結果是`105`和`3`。而“後行斷言”時，由於執行順序是從右到左，第二個括號是貪婪模式，第一個括號只能捕獲一個字元，所以結果是`1`和`053`。

其次，“後行斷言”的反斜槓引用，也與通常的順序相反，必須放在對應的那個括號之前。

```javascript
/(?<=(o)d\1)r/.exec('hodor')  // null
/(?<=\1d(o))r/.exec('hodor')  // ["r", "o"]
```

上面程式碼中，如果後行斷言的反斜槓引用（`\1`）放在括號的後面，就不會得到匹配結果，必須放在前面才可以。因為後行斷言是先從左到右掃描，發現匹配以後再回過頭，從右到左完成反斜槓引用。

## Unicode 屬性類

ES2018 [引入](https://github.com/tc39/proposal-regexp-unicode-property-escapes)了一種新的類的寫法`\p{...}`和`\P{...}`，允許正則表示式匹配符合 Unicode 某種屬性的所有字元。

```javascript
const regexGreekSymbol = /\p{Script=Greek}/u;
regexGreekSymbol.test('π') // true
```

上面程式碼中，`\p{Script=Greek}`指定匹配一個希臘文字母，所以匹配`π`成功。

Unicode 屬性類要指定屬性名和屬性值。

```javascript
\p{UnicodePropertyName=UnicodePropertyValue}
```

對於某些屬性，可以只寫屬性名，或者只寫屬性值。

```javascript
\p{UnicodePropertyName}
\p{UnicodePropertyValue}
```

`\P{…}`是`\p{…}`的反向匹配，即匹配不滿足條件的字元。

注意，這兩種類只對 Unicode 有效，所以使用的時候一定要加上`u`修飾符。如果不加`u`修飾符，正則表示式使用`\p`和`\P`會報錯，ECMAScript 預留了這兩個類。

由於 Unicode 的各種屬性非常多，所以這種新的類的表達能力非常強。

```javascript
const regex = /^\p{Decimal_Number}+$/u;
regex.test('𝟏𝟐𝟑𝟜𝟝𝟞𝟩𝟪𝟫𝟬𝟭𝟮𝟯𝟺𝟻𝟼') // true
```

上面程式碼中，屬性類指定匹配所有十進位制字元，可以看到各種字型的十進位制字元都會匹配成功。

`\p{Number}`甚至能匹配羅馬數字。

```javascript
// 匹配所有數字
const regex = /^\p{Number}+$/u;
regex.test('²³¹¼½¾') // true
regex.test('㉛㉜㉝') // true
regex.test('ⅠⅡⅢⅣⅤⅥⅦⅧⅨⅩⅪⅫ') // true
```

下面是其他一些例子。

```javascript
// 匹配所有空格
\p{White_Space}

// 匹配各種文字的所有字母，等同於 Unicode 版的 \w
[\p{Alphabetic}\p{Mark}\p{Decimal_Number}\p{Connector_Punctuation}\p{Join_Control}]

// 匹配各種文字的所有非字母的字元，等同於 Unicode 版的 \W
[^\p{Alphabetic}\p{Mark}\p{Decimal_Number}\p{Connector_Punctuation}\p{Join_Control}]

// 匹配 Emoji
/\p{Emoji_Modifier_Base}\p{Emoji_Modifier}?|\p{Emoji_Presentation}|\p{Emoji}\uFE0F/gu

// 匹配所有的箭頭字元
const regexArrows = /^\p{Block=Arrows}+$/u;
regexArrows.test('←↑→↓↔↕↖↗↘↙⇏⇐⇑⇒⇓⇔⇕⇖⇗⇘⇙⇧⇩') // true
```

## 具名組匹配

### 簡介

正則表示式使用圓括號進行組匹配。

```javascript
const RE_DATE = /(\d{4})-(\d{2})-(\d{2})/;
```

上面程式碼中，正則表示式裡面有三組圓括號。使用`exec`方法，就可以將這三組匹配結果提取出來。

```javascript
const RE_DATE = /(\d{4})-(\d{2})-(\d{2})/;

const matchObj = RE_DATE.exec('1999-12-31');
const year = matchObj[1]; // 1999
const month = matchObj[2]; // 12
const day = matchObj[3]; // 31
```

組匹配的一個問題是，每一組的匹配含義不容易看出來，而且只能用數字序號（比如`matchObj[1]`）引用，要是組的順序變了，引用的時候就必須修改序號。

ES2018 引入了[具名組匹配](https://github.com/tc39/proposal-regexp-named-groups)（Named Capture Groups），允許為每一個組匹配指定一個名字，既便於閱讀程式碼，又便於引用。

```javascript
const RE_DATE = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;

const matchObj = RE_DATE.exec('1999-12-31');
const year = matchObj.groups.year; // "1999"
const month = matchObj.groups.month; // "12"
const day = matchObj.groups.day; // "31"
```

上面程式碼中，“具名組匹配”在圓括號內部，模式的頭部新增“問號 + 尖括號 + 組名”（`?<year>`），然後就可以在`exec`方法返回結果的`groups`屬性上引用該組名。同時，數字序號（`matchObj[1]`）依然有效。

具名組匹配等於為每一組匹配加上了 ID，便於描述匹配的目的。如果組的順序變了，也不用改變匹配後的處理程式碼。

如果具名組沒有匹配，那麼對應的`groups`物件屬性會是`undefined`。

```javascript
const RE_OPT_A = /^(?<as>a+)?$/;
const matchObj = RE_OPT_A.exec('');

matchObj.groups.as // undefined
'as' in matchObj.groups // true
```

上面程式碼中，具名組`as`沒有找到匹配，那麼`matchObj.groups.as`屬性值就是`undefined`，並且`as`這個鍵名在`groups`是始終存在的。

### 解構賦值和替換

有了具名組匹配以後，可以使用解構賦值直接從匹配結果上為變數賦值。

```javascript
let {groups: {one, two}} = /^(?<one>.*):(?<two>.*)$/u.exec('foo:bar');
one  // foo
two  // bar
```

字串替換時，使用`$<組名>`引用具名組。

```javascript
let re = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/u;

'2015-01-02'.replace(re, '$<day>/$<month>/$<year>')
// '02/01/2015'
```

上面程式碼中，`replace`方法的第二個引數是一個字串，而不是正則表示式。

`replace`方法的第二個引數也可以是函式，該函式的引數序列如下。

```javascript
'2015-01-02'.replace(re, (
   matched, // 整個匹配結果 2015-01-02
   capture1, // 第一個組匹配 2015
   capture2, // 第二個組匹配 01
   capture3, // 第三個組匹配 02
   position, // 匹配開始的位置 0
   S, // 原字串 2015-01-02
   groups // 具名組構成的一個物件 {year, month, day}
 ) => {
 let {day, month, year} = groups;
 return `${day}/${month}/${year}`;
});
```

具名組匹配在原來的基礎上，新增了最後一個函式引數：具名組構成的一個物件。函式內部可以直接對這個物件進行解構賦值。

### 引用

如果要在正則表示式內部引用某個“具名組匹配”，可以使用`\k<組名>`的寫法。

```javascript
const RE_TWICE = /^(?<word>[a-z]+)!\k<word>$/;
RE_TWICE.test('abc!abc') // true
RE_TWICE.test('abc!ab') // false
```

數字引用（`\1`）依然有效。

```javascript
const RE_TWICE = /^(?<word>[a-z]+)!\1$/;
RE_TWICE.test('abc!abc') // true
RE_TWICE.test('abc!ab') // false
```

這兩種引用語法還可以同時使用。

```javascript
const RE_TWICE = /^(?<word>[a-z]+)!\k<word>!\1$/;
RE_TWICE.test('abc!abc!abc') // true
RE_TWICE.test('abc!abc!ab') // false
```

## 正則匹配索引

正則匹配結果的開始位置和結束位置，目前獲取並不是很方便。正則例項的`exec()`方法，返回結果有一個`index`屬性，可以獲取整個匹配結果的開始位置，但是如果包含組匹配，每個組匹配的開始位置，很難拿到。

現在有一個[第三階段提案](https://github.com/tc39/proposal-regexp-match-Indices)，為`exec()`方法的返回結果加上`indices`屬性，在這個屬性上面可以拿到匹配的開始位置和結束位置。

```javascript
const text = 'zabbcdef';
const re = /ab/;
const result = re.exec(text);

result.index // 1
result.indices // [ [1, 3] ]
```

上面例子中，`exec()`方法的返回結果`result`，它的`index`屬性是整個匹配結果（`ab`）的開始位置，而它的`indices`屬性是一個數組，成員是每個匹配的開始位置和結束位置的陣列。由於該例子的正則表示式沒有組匹配，所以`indices`陣列只有一個成員，表示整個匹配的開始位置是`1`，結束位置是`3`。

注意，開始位置包含在匹配結果之中，但是結束位置不包含在匹配結果之中。比如，匹配結果為`ab`，分別是原始字串的第1位和第2位，那麼結束位置就是第3位。

如果正則表示式包含組匹配，那麼`indices`屬性對應的陣列就會包含多個成員，提供每個組匹配的開始位置和結束位置。

```javascript
const text = 'zabbcdef';
const re = /ab+(cd)/;
const result = re.exec(text);

result.indices // [ [ 1, 6 ], [ 4, 6 ] ]
```

上面例子中，正則表示式包含一個組匹配，那麼`indices`屬性陣列就有兩個成員，第一個成員是整個匹配結果（`abbcd`）的開始位置和結束位置，第二個成員是組匹配（`cd`）的開始位置和結束位置。

下面是多個組匹配的例子。

```javascript
const text = 'zabbcdef';
const re = /ab+(cd(ef))/;
const result = re.exec(text);

result.indices // [ [1, 8], [4, 8], [6, 8] ]
```

上面例子中，正則表示式包含兩個組匹配，所以`indices`屬性陣列就有三個成員。

如果正則表示式包含具名組匹配，`indices`屬性陣列還會有一個`groups`屬性。該屬性是一個物件，可以從該物件獲取具名組匹配的開始位置和結束位置。

```javascript
const text = 'zabbcdef';
const re = /ab+(?<Z>cd)/;
const result = re.exec(text);

result.indices.groups // { Z: [ 4, 6 ] }
```

上面例子中，`exec()`方法返回結果的`indices.groups`屬性是一個物件，提供具名組匹配`Z`的開始位置和結束位置。

如果獲取組匹配不成功，`indices`屬性陣列的對應成員則為`undefined`，`indices.groups`屬性物件的對應成員也是`undefined`。

```javascript
const text = 'zabbcdef';
const re = /ab+(?<Z>ce)?/;
const result = re.exec(text);

result.indices[1] // undefined
result.indices.groups['Z'] // undefined
```

上面例子中，由於組匹配不成功，所以`indices`屬性陣列和`indices.groups`屬性物件對應的組匹配成員都是`undefined`。

## String.prototype.matchAll()

如果一個正則表示式在字串裡面有多個匹配，現在一般使用`g`修飾符或`y`修飾符，在迴圈裡面逐一取出。

```javascript
var regex = /t(e)(st(\d?))/g;
var string = 'test1test2test3';

var matches = [];
var match;
while (match = regex.exec(string)) {
  matches.push(match);
}

matches
// [
//   ["test1", "e", "st1", "1", index: 0, input: "test1test2test3"],
//   ["test2", "e", "st2", "2", index: 5, input: "test1test2test3"],
//   ["test3", "e", "st3", "3", index: 10, input: "test1test2test3"]
// ]
```

上面程式碼中，`while`迴圈取出每一輪的正則匹配，一共三輪。

[ES2020](https://github.com/tc39/proposal-string-matchall) 增加了`String.prototype.matchAll()`方法，可以一次性取出所有匹配。不過，它返回的是一個遍歷器（Iterator），而不是陣列。

```javascript
const string = 'test1test2test3';
const regex = /t(e)(st(\d?))/g;

for (const match of string.matchAll(regex)) {
  console.log(match);
}
// ["test1", "e", "st1", "1", index: 0, input: "test1test2test3"]
// ["test2", "e", "st2", "2", index: 5, input: "test1test2test3"]
// ["test3", "e", "st3", "3", index: 10, input: "test1test2test3"]
```

上面程式碼中，由於`string.matchAll(regex)`返回的是遍歷器，所以可以用`for...of`迴圈取出。相對於返回陣列，返回遍歷器的好處在於，如果匹配結果是一個很大的陣列，那麼遍歷器比較節省資源。

遍歷器轉為陣列是非常簡單的，使用`...`運算子和`Array.from()`方法就可以了。

```javascript
// 轉為陣列的方法一
[...string.matchAll(regex)]

// 轉為陣列的方法二
Array.from(string.matchAll(regex))
```

