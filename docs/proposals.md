# 最新提案

本章介紹一些尚未進入標準、但很有希望的最新提案。

## do 表示式

本質上，塊級作用域是一個語句，將多個操作封裝在一起，沒有返回值。

```javascript
{
  let t = f();
  t = t * t + 1;
}
```

上面程式碼中，塊級作用域將兩個語句封裝在一起。但是，在塊級作用域以外，沒有辦法得到`t`的值，因為塊級作用域不返回值，除非`t`是全域性變數。

現在有一個[提案](https://github.com/tc39/proposal-do-expressions)，使得塊級作用域可以變為表示式，也就是說可以返回值，辦法就是在塊級作用域之前加上`do`，使它變為`do`表示式，然後就會返回內部最後執行的表示式的值。

```javascript
let x = do {
  let t = f();
  t * t + 1;
};
```

上面程式碼中，變數`x`會得到整個塊級作用域的返回值（`t * t + 1`）。

`do`表示式的邏輯非常簡單：封裝的是什麼，就會返回什麼。

```javascript
// 等同於 <表示式>
do { <表示式>; }

// 等同於 <語句>
do { <語句> }
```

`do`表示式的好處是可以封裝多個語句，讓程式更加模組化，就像樂高積木那樣一塊塊拼裝起來。

```javascript
let x = do {
  if (foo()) { f() }
  else if (bar()) { g() }
  else { h() }
};
```

上面程式碼的本質，就是根據函式`foo`的執行結果，呼叫不同的函式，將返回結果賦給變數`x`。使用`do`表示式，就將這個操作的意圖表達得非常簡潔清晰。而且，`do`塊級作用域提供了單獨的作用域，內部操作可以與全域性作用域隔絕。

值得一提的是，`do`表示式在 JSX 語法中非常好用。

```javascript
return (
  <nav>
    <Home />
    {
      do {
        if (loggedIn) {
          <LogoutButton />
        } else {
          <LoginButton />
        }
      }
    }
  </nav>
)
```

上面程式碼中，如果不用`do`表示式，就只能用三元判斷運算子（`?:`）。那樣的話，一旦判斷邏輯複雜，程式碼就會變得很不易讀。

## throw 表示式

JavaScript 語法規定`throw`是一個命令，用來丟擲錯誤，不能用於表示式之中。

```javascript
// 報錯
console.log(throw new Error());
```

上面程式碼中，`console.log`的引數必須是一個表示式，如果是一個`throw`語句就會報錯。

現在有一個[提案](https://github.com/tc39/proposal-throw-expressions)，允許`throw`用於表示式。

```javascript
// 引數的預設值
function save(filename = throw new TypeError("Argument required")) {
}

// 箭頭函式的返回值
lint(ast, {
  with: () => throw new Error("avoid using 'with' statements.")
});

// 條件表示式
function getEncoder(encoding) {
  const encoder = encoding === "utf8" ?
    new UTF8Encoder() :
    encoding === "utf16le" ?
      new UTF16Encoder(false) :
      encoding === "utf16be" ?
        new UTF16Encoder(true) :
        throw new Error("Unsupported encoding");
}

// 邏輯表示式
class Product {
  get id() {
    return this._id;
  }
  set id(value) {
    this._id = value || throw new Error("Invalid value");
  }
}
```

上面程式碼中，`throw`都出現在表示式裡面。

語法上，`throw`表示式裡面的`throw`不再是一個命令，而是一個運算子。為了避免與`throw`命令混淆，規定`throw`出現在行首，一律解釋為`throw`語句，而不是`throw`表示式。

## 函式的部分執行

### 語法

多引數的函式有時需要繫結其中的一個或多個引數，然後返回一個新函式。

```javascript
function add(x, y) { return x + y; }
function add7(x) { return x + 7; }
```

上面程式碼中，`add7`函式其實是`add`函式的一個特殊版本，透過將一個引數繫結為`7`，就可以從`add`得到`add7`。

```javascript
// bind 方法
const add7 = add.bind(null, 7);

// 箭頭函式
const add7 = x => add(x, 7);
```

上面兩種寫法都有些冗餘。其中，`bind`方法的侷限更加明顯，它必須提供`this`，並且只能從前到後一個個繫結引數，無法只繫結非頭部的引數。

現在有一個[提案](https://github.com/tc39/proposal-partial-application)，使得繫結引數並返回一個新函式更加容易。這叫做函式的部分執行（partial application）。

```javascript
const add = (x, y) => x + y;
const addOne = add(1, ?);

const maxGreaterThanZero = Math.max(0, ...);
```

根據新提案，`?`是單個引數的佔位符，`...`是多個引數的佔位符。以下的形式都屬於函式的部分執行。

```javascript
f(x, ?)
f(x, ...)
f(?, x)
f(..., x)
f(?, x, ?)
f(..., x, ...)
```

`?`和`...`只能出現在函式的呼叫之中，並且會返回一個新函式。

```javascript
const g = f(?, 1, ...);
// 等同於
const g = (x, ...y) => f(x, 1, ...y);
```

函式的部分執行，也可以用於物件的方法。

```javascript
let obj = {
  f(x, y) { return x + y; },
};

const g = obj.f(?, 3);
g(1) // 4
```

### 注意點

函式的部分執行有一些特別注意的地方。

（1）函式的部分執行是基於原函式的。如果原函式發生變化，部分執行生成的新函式也會立即反映這種變化。

```javascript
let f = (x, y) => x + y;

const g = f(?, 3);
g(1); // 4

// 替換函式 f
f = (x, y) => x * y;

g(1); // 3
```

上面程式碼中，定義了函式的部分執行以後，更換原函式會立即影響到新函式。

（2）如果預先提供的那個值是一個表示式，那麼這個表示式並不會在定義時求值，而是在每次呼叫時求值。

```javascript
let a = 3;
const f = (x, y) => x + y;

const g = f(?, a);
g(1); // 4

// 改變 a 的值
a = 10;
g(1); // 11
```

上面程式碼中，預先提供的引數是變數`a`，那麼每次呼叫函式`g`的時候，才會對`a`進行求值。

（3）如果新函式的引數多於佔位符的數量，那麼多餘的引數將被忽略。

```javascript
const f = (x, ...y) => [x, ...y];
const g = f(?, 1);
g(2, 3, 4); // [2, 1]
```

上面程式碼中，函式`g`只有一個佔位符，也就意味著它只能接受一個引數，多餘的引數都會被忽略。

寫成下面這樣，多餘的引數就沒有問題。

```javascript
const f = (x, ...y) => [x, ...y];
const g = f(?, 1, ...);
g(2, 3, 4); // [2, 1, 3, 4];
```

（4）`...`只會被採集一次，如果函式的部分執行使用了多個`...`，那麼每個`...`的值都將相同。

```javascript
const f = (...x) => x;
const g = f(..., 9, ...);
g(1, 2, 3); // [1, 2, 3, 9, 1, 2, 3]
```

上面程式碼中，`g`定義了兩個`...`佔位符，真正執行的時候，它們的值是一樣的。

## 管道運算子

Unix 作業系統有一個管道機制（pipeline），可以把前一個操作的值傳給後一個操作。這個機制非常有用，使得簡單的操作可以組合成為複雜的操作。許多語言都有管道的實現，現在有一個[提案](https://github.com/tc39/proposal-pipeline-operator)，讓 JavaScript 也擁有管道機制。

JavaScript 的管道是一個運算子，寫作`|>`。它的左邊是一個表示式，右邊是一個函式。管道運算子把左邊表示式的值，傳入右邊的函式進行求值。

```javascript
x |> f
// 等同於
f(x)
```

管道運算子最大的好處，就是可以把巢狀的函式，寫成從左到右的鏈式表示式。

```javascript
function doubleSay (str) {
  return str + ", " + str;
}

function capitalize (str) {
  return str[0].toUpperCase() + str.substring(1);
}

function exclaim (str) {
  return str + '!';
}
```

上面是三個簡單的函式。如果要巢狀執行，傳統的寫法和管道的寫法分別如下。

```javascript
// 傳統的寫法
exclaim(capitalize(doubleSay('hello')))
// "Hello, hello!"

// 管道的寫法
'hello'
  |> doubleSay
  |> capitalize
  |> exclaim
// "Hello, hello!"
```

管道運算子只能傳遞一個值，這意味著它右邊的函式必須是一個單引數函式。如果是多引數函式，就必須進行柯里化，改成單引數的版本。

```javascript
function double (x) { return x + x; }
function add (x, y) { return x + y; }

let person = { score: 25 };
person.score
  |> double
  |> (_ => add(7, _))
// 57
```

上面程式碼中，`add`函式需要兩個引數。但是，管道運算子只能傳入一個值，因此需要事先提供另一個引數，並將其改成單引數的箭頭函式`_ => add(7, _)`。這個函式裡面的下劃線並沒有特別的含義，可以用其他符號代替，使用下劃線只是因為，它能夠形象地表示這裡是佔位符。

管道運算子對於`await`函式也適用。

```javascript
x |> await f
// 等同於
await f(x)

const userAge = userId |> await fetchUserById |> getAgeFromUser;
// 等同於
const userAge = getAgeFromUser(await fetchUserById(userId));
```

## 數值分隔符

歐美語言中，較長的數值允許每三位新增一個分隔符（通常是一個逗號），增加數值的可讀性。比如，`1000`可以寫作`1,000`。

現在有一個[提案](https://github.com/tc39/proposal-numeric-separator)，允許 JavaScript 的數值使用下劃線（`_`）作為分隔符。

```javascript
let budget = 1_000_000_000_000;
budget === 10 ** 12 // true
```

JavaScript 的數值分隔符沒有指定間隔的位數，也就是說，可以每三位新增一個分隔符，也可以每一位、每兩位、每四位新增一個。

```javascript
123_00 === 12_300 // true

12345_00 === 123_4500 // true
12345_00 === 1_234_500 // true
```

小數和科學計數法也可以使用數值分隔符。

```javascript
// 小數
0.000_001
// 科學計數法
1e10_000
```

數值分隔符有幾個使用注意點。

- 不能在數值的最前面（leading）或最後面（trailing）。
- 不能兩個或兩個以上的分隔符連在一起。
- 小數點的前後不能有分隔符。
- 科學計數法裡面，表示指數的`e`或`E`前後不能有分隔符。

下面的寫法都會報錯。

```javascript
// 全部報錯
3_.141
3._141
1_e12
1e_12
123__456
_1464301
1464301_
```

除了十進位制，其他進位制的數值也可以使用分隔符。

```javascript
// 二進位制
0b1010_0001_1000_0101
// 十六進位制
0xA0_B0_C0
```

注意，分隔符不能緊跟著進位制的字首`0b`、`0B`、`0o`、`0O`、`0x`、`0X`。

```javascript
// 報錯
0_b111111000
0b_111111000
```

下面三個將字串轉成數值的函式，不支援數值分隔符。主要原因是提案的設計者認為，數值分隔符主要是為了編碼時書寫數值的方便，而不是為了處理外部輸入的資料。

- Number()
- parseInt()
- parseFloat()

```javascript
Number('123_456') // NaN
parseInt('123_456') // 123
```

## Math.signbit()

`Math.sign()`用來判斷一個值的正負，但是如果引數是`-0`，它會返回`-0`。

```javascript
Math.sign(-0) // -0
```

這導致對於判斷符號位的正負，`Math.sign()`不是很有用。JavaScript 內部使用 64 位浮點數（國際標準 IEEE 754）表示數值，IEEE 754 規定第一位是符號位，`0`表示正數，`1`表示負數。所以會有兩種零，`+0`是符號位為`0`時的零值，`-0`是符號位為`1`時的零值。實際程式設計中，判斷一個值是`+0`還是`-0`非常麻煩，因為它們是相等的。

```javascript
+0 === -0 // true
```

目前，有一個[提案](http://jfbastien.github.io/papers/Math.signbit.html)，引入了`Math.signbit()`方法判斷一個數的符號位是否設定了。

```javascript
Math.signbit(2) //false
Math.signbit(-2) //true
Math.signbit(0) //false
Math.signbit(-0) //true
```

可以看到，該方法正確返回了`-0`的符號位是設定了的。

該方法的演算法如下。

- 如果引數是`NaN`，返回`false`
- 如果引數是`-0`，返回`true`
- 如果引數是負值，返回`true`
- 其他情況返回`false`

## 雙冒號運算子

箭頭函式可以繫結`this`物件，大大減少了顯式繫結`this`物件的寫法（`call`、`apply`、`bind`）。但是，箭頭函式並不適用於所有場合，所以現在有一個[提案](https://github.com/zenparsing/es-function-bind)，提出了“函式繫結”（function bind）運算子，用來取代`call`、`apply`、`bind`呼叫。

函式繫結運算子是並排的兩個冒號（`::`），雙冒號左邊是一個物件，右邊是一個函式。該運算子會自動將左邊的物件，作為上下文環境（即`this`物件），繫結到右邊的函式上面。

```javascript
foo::bar;
// 等同於
bar.bind(foo);

foo::bar(...arguments);
// 等同於
bar.apply(foo, arguments);

const hasOwnProperty = Object.prototype.hasOwnProperty;
function hasOwn(obj, key) {
  return obj::hasOwnProperty(key);
}
```

如果雙冒號左邊為空，右邊是一個物件的方法，則等於將該方法繫結在該物件上面。

```javascript
var method = obj::obj.foo;
// 等同於
var method = ::obj.foo;

let log = ::console.log;
// 等同於
var log = console.log.bind(console);
```

如果雙冒號運算子的運算結果，還是一個物件，就可以採用鏈式寫法。

```javascript
import { map, takeWhile, forEach } from "iterlib";

getPlayers()
::map(x => x.character())
::takeWhile(x => x.strength > 100)
::forEach(x => console.log(x));
```

## Realm API

[Realm API](https://github.com/tc39/proposal-realms) 提供沙箱功能（sandbox），允許隔離程式碼，防止那些被隔離的程式碼拿到全域性物件。

以前，經常使用`<iframe>`作為沙箱。

```javascript
const globalOne = window;
let iframe = document.createElement('iframe');
document.body.appendChild(iframe);
const globalTwo = iframe.contentWindow;
```

上面程式碼中，`<iframe>`的全域性物件是獨立的（`iframe.contentWindow`）。Realm API 可以取代這個功能。

```javascript
const globalOne = window;
const globalTwo = new Realm().global;
```

上面程式碼中，`Realm API`單獨提供了一個全域性物件`new Realm().global`。

Realm API 提供一個`Realm()`建構函式，用來生成一個 Realm 物件。該物件的`global`屬性指向一個新的頂層物件，這個頂層物件跟原始的頂層物件類似。

```javascript
const globalOne = window;
const globalTwo = new Realm().global;

globalOne.evaluate('1 + 2') // 3
globalTwo.evaluate('1 + 2') // 3
```

上面程式碼中，Realm 生成的頂層物件的`evaluate()`方法，可以執行程式碼。

下面的程式碼可以證明，Realm 頂層物件與原始頂層物件是兩個物件。

```javascript
let a1 = globalOne.evaluate('[1,2,3]');
let a2 = globalTwo.evaluate('[1,2,3]');
a1.prototype === a2.prototype; // false
a1 instanceof globalTwo.Array; // false
a2 instanceof globalOne.Array; // false
```

上面程式碼中，Realm 沙箱裡面的陣列的原型物件，跟原始環境裡面的陣列是不一樣的。

Realm 沙箱裡面只能執行 ECMAScript 語法提供的 API，不能執行宿主環境提供的 API。

```javascript
globalTwo.evaluate('console.log(1)')
// throw an error: console is undefined
```

上面程式碼中，Realm 沙箱裡面沒有`console`物件，導致報錯。因為`console`不是語法標準，是宿主環境提供的。

如果要解決這個問題，可以使用下面的程式碼。

```javascript
globalTwo.console = globalOne.console;
```

`Realm()`建構函式可以接受一個引數物件，該引數物件的`intrinsics`屬性可以指定 Realm 沙箱繼承原始頂層物件的方法。

```javascript
const r1 = new Realm();
r1.global === this;
r1.global.JSON === JSON; // false

const r2 = new Realm({ intrinsics: 'inherit' });
r2.global === this; // false
r2.global.JSON === JSON; // true
```

上面程式碼中，正常情況下，沙箱的`JSON`方法不同於原始的`JSON`物件。但是，`Realm()`建構函式接受`{ intrinsics: 'inherit' }`作為引數以後，就會繼承原始頂層物件的方法。

使用者可以自己定義`Realm`的子類，用來定製自己的沙箱。

```javascript
class FakeWindow extends Realm {
  init() {
    super.init();
    let global = this.global;

    global.document = new FakeDocument(...);
    global.alert = new Proxy(fakeAlert, { ... });
    // ...
  }
}
```

上面程式碼中，`FakeWindow`模擬了一個假的頂層物件`window`。

## `#!`命令

Unix 的命令列指令碼都支援`#!`命令，又稱為 Shebang 或 Hashbang。這個命令放在指令碼的第一行，用來指定指令碼的執行器。

比如 Bash 指令碼的第一行。

```bash
#!/bin/sh
```

Python 指令碼的第一行。

```python
#!/usr/bin/env python
```

現在有一個[提案](https://github.com/tc39/proposal-hashbang)，為 JavaScript 指令碼引入了`#!`命令，寫在指令碼檔案或者模組檔案的第一行。

```javascript
// 寫在指令碼檔案第一行
#!/usr/bin/env node
'use strict';
console.log(1);

// 寫在模組檔案第一行
#!/usr/bin/env node
export {};
console.log(1);
```

有了這一行以後，Unix 命令列就可以直接執行指令碼。

```bash
# 以前執行指令碼的方式
$ node hello.js

# hashbang 的方式
$ ./hello.js
```

對於 JavaScript 引擎來說，會把`#!`理解成註釋，忽略掉這一行。

## import.meta

開發者使用一個模組時，有時需要知道模板本身的一些資訊（比如模組的路徑）。現在有一個[提案](https://github.com/tc39/proposal-import-meta)，為 import 命令添加了一個元屬性`import.meta`，返回當前模組的元資訊。

`import.meta`只能在模組內部使用，如果在模組外部使用會報錯。

這個屬性返回一個物件，該物件的各種屬性就是當前執行的指令碼的元資訊。具體包含哪些屬性，標準沒有規定，由各個執行環境自行決定。一般來說，`import.meta`至少會有下面兩個屬性。

**（1）import.meta.url**

`import.meta.url`返回當前模組的 URL 路徑。舉例來說，當前模組主檔案的路徑是`https://foo.com/main.js`，`import.meta.url`就返回這個路徑。如果模組裡面還有一個數據檔案`data.txt`，那麼就可以用下面的程式碼，獲取這個資料檔案的路徑。

```javascript
new URL('data.txt', import.meta.url)
```

注意，Node.js 環境中，`import.meta.url`返回的總是本地路徑，即是`file:URL`協議的字串，比如`file:///home/user/foo.js`。

**（2）import.meta.scriptElement**

`import.meta.scriptElement`是瀏覽器特有的元屬性，返回載入模組的那個`<script>`元素，相當於`document.currentScript`屬性。

```javascript
// HTML 程式碼為
// <script type="module" src="my-module.js" data-foo="abc"></script>

// my-module.js 內部執行下面的程式碼
import.meta.scriptElement.dataset.foo
// "abc"
```

