# let 和 const 命令

## let 命令

### 基本用法

ES6 新增了`let`命令，用來宣告變數。它的用法類似於`var`，但是所宣告的變數，只在`let`命令所在的程式碼塊內有效。

```javascript
{
  let a = 10;
  var b = 1;
}

a // ReferenceError: a is not defined.
b // 1
```

上面程式碼在程式碼塊之中，分別用`let`和`var`聲明瞭兩個變數。然後在程式碼塊之外呼叫這兩個變數，結果`let`宣告的變數報錯，`var`宣告的變數返回了正確的值。這表明，`let`宣告的變數只在它所在的程式碼塊有效。

`for`迴圈的計數器，就很合適使用`let`命令。

```javascript
for (let i = 0; i < 10; i++) {
  // ...
}

console.log(i);
// ReferenceError: i is not defined
```

上面程式碼中，計數器`i`只在`for`迴圈體內有效，在迴圈體外引用就會報錯。

下面的程式碼如果使用`var`，最後輸出的是`10`。

```javascript
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 10
```

上面程式碼中，變數`i`是`var`命令宣告的，在全域性範圍內都有效，所以全域性只有一個變數`i`。每一次迴圈，變數`i`的值都會發生改變，而迴圈內被賦給陣列`a`的函式內部的`console.log(i)`，裡面的`i`指向的就是全域性的`i`。也就是說，所有陣列`a`的成員裡面的`i`，指向的都是同一個`i`，導致執行時輸出的是最後一輪的`i`的值，也就是 10。

如果使用`let`，宣告的變數僅在塊級作用域內有效，最後輸出的是 6。

```javascript
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6
```

上面程式碼中，變數`i`是`let`宣告的，當前的`i`只在本輪迴圈有效，所以每一次迴圈的`i`其實都是一個新的變數，所以最後輸出的是`6`。你可能會問，如果每一輪迴圈的變數`i`都是重新宣告的，那它怎麼知道上一輪迴圈的值，從而計算出本輪迴圈的值？這是因為 JavaScript 引擎內部會記住上一輪迴圈的值，初始化本輪的變數`i`時，就在上一輪迴圈的基礎上進行計算。

另外，`for`迴圈還有一個特別之處，就是設定迴圈變數的那部分是一個父作用域，而迴圈體內部是一個單獨的子作用域。

```javascript
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i);
}
// abc
// abc
// abc
```

上面程式碼正確執行，輸出了 3 次`abc`。這表明函式內部的變數`i`與迴圈變數`i`不在同一個作用域，有各自單獨的作用域。

### 不存在變數提升

`var`命令會發生“變數提升”現象，即變數可以在宣告之前使用，值為`undefined`。這種現象多多少少是有些奇怪的，按照一般的邏輯，變數應該在宣告語句之後才可以使用。

為了糾正這種現象，`let`命令改變了語法行為，它所宣告的變數一定要在聲明後使用，否則報錯。

```javascript
// var 的情況
console.log(foo); // 輸出undefined
var foo = 2;

// let 的情況
console.log(bar); // 報錯ReferenceError
let bar = 2;
```

上面程式碼中，變數`foo`用`var`命令宣告，會發生變數提升，即指令碼開始執行時，變數`foo`已經存在了，但是沒有值，所以會輸出`undefined`。變數`bar`用`let`命令宣告，不會發生變數提升。這表示在宣告它之前，變數`bar`是不存在的，這時如果用到它，就會丟擲一個錯誤。

### 暫時性死區

只要塊級作用域記憶體在`let`命令，它所宣告的變數就“繫結”（binding）這個區域，不再受外部的影響。

```javascript
var tmp = 123;

if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
}
```

上面程式碼中，存在全域性變數`tmp`，但是塊級作用域內`let`又聲明瞭一個區域性變數`tmp`，導致後者繫結這個塊級作用域，所以在`let`宣告變數前，對`tmp`賦值會報錯。

ES6 明確規定，如果區塊中存在`let`和`const`命令，這個區塊對這些命令宣告的變數，從一開始就形成了封閉作用域。凡是在宣告之前就使用這些變數，就會報錯。

總之，在程式碼塊內，使用`let`命令宣告變數之前，該變數都是不可用的。這在語法上，稱為“暫時性死區”（temporal dead zone，簡稱 TDZ）。

```javascript
if (true) {
  // TDZ開始
  tmp = 'abc'; // ReferenceError
  console.log(tmp); // ReferenceError

  let tmp; // TDZ結束
  console.log(tmp); // undefined

  tmp = 123;
  console.log(tmp); // 123
}
```

上面程式碼中，在`let`命令宣告變數`tmp`之前，都屬於變數`tmp`的“死區”。

“暫時性死區”也意味著`typeof`不再是一個百分之百安全的操作。

```javascript
typeof x; // ReferenceError
let x;
```

上面程式碼中，變數`x`使用`let`命令宣告，所以在宣告之前，都屬於`x`的“死區”，只要用到該變數就會報錯。因此，`typeof`執行時就會丟擲一個`ReferenceError`。

作為比較，如果一個變數根本沒有被宣告，使用`typeof`反而不會報錯。

```javascript
typeof undeclared_variable // "undefined"
```

上面程式碼中，`undeclared_variable`是一個不存在的變數名，結果返回“undefined”。所以，在沒有`let`之前，`typeof`運算子是百分之百安全的，永遠不會報錯。現在這一點不成立了。這樣的設計是為了讓大家養成良好的程式設計習慣，變數一定要在宣告之後使用，否則就報錯。

有些“死區”比較隱蔽，不太容易發現。

```javascript
function bar(x = y, y = 2) {
  return [x, y];
}

bar(); // 報錯
```

上面程式碼中，呼叫`bar`函式之所以報錯（某些實現可能不報錯），是因為引數`x`預設值等於另一個引數`y`，而此時`y`還沒有宣告，屬於“死區”。如果`y`的預設值是`x`，就不會報錯，因為此時`x`已經聲明瞭。

```javascript
function bar(x = 2, y = x) {
  return [x, y];
}
bar(); // [2, 2]
```

另外，下面的程式碼也會報錯，與`var`的行為不同。

```javascript
// 不報錯
var x = x;

// 報錯
let x = x;
// ReferenceError: x is not defined
```

上面程式碼報錯，也是因為暫時性死區。使用`let`宣告變數時，只要變數在還沒有宣告完成前使用，就會報錯。上面這行就屬於這個情況，在變數`x`的宣告語句還沒有執行完成前，就去取`x`的值，導致報錯”x 未定義“。

ES6 規定暫時性死區和`let`、`const`語句不出現變數提升，主要是為了減少執行時錯誤，防止在變數宣告前就使用這個變數，從而導致意料之外的行為。這樣的錯誤在 ES5 是很常見的，現在有了這種規定，避免此類錯誤就很容易了。

總之，暫時性死區的本質就是，只要一進入當前作用域，所要使用的變數就已經存在了，但是不可獲取，只有等到宣告變數的那一行程式碼出現，才可以獲取和使用該變數。

### 不允許重複宣告

`let`不允許在相同作用域內，重複宣告同一個變數。

```javascript
// 報錯
function func() {
  let a = 10;
  var a = 1;
}

// 報錯
function func() {
  let a = 10;
  let a = 1;
}
```

因此，不能在函式內部重新宣告引數。

```javascript
function func(arg) {
  let arg;
}
func() // 報錯

function func(arg) {
  {
    let arg;
  }
}
func() // 不報錯
```

## 塊級作用域

### 為什麼需要塊級作用域？

ES5 只有全域性作用域和函式作用域，沒有塊級作用域，這帶來很多不合理的場景。

第一種場景，內層變數可能會覆蓋外層變數。

```javascript
var tmp = new Date();

function f() {
  console.log(tmp);
  if (false) {
    var tmp = 'hello world';
  }
}

f(); // undefined
```

上面程式碼的原意是，`if`程式碼塊的外部使用外層的`tmp`變數，內部使用內層的`tmp`變數。但是，函式`f`執行後，輸出結果為`undefined`，原因在於變數提升，導致內層的`tmp`變數覆蓋了外層的`tmp`變數。

第二種場景，用來計數的迴圈變數洩露為全域性變數。

```javascript
var s = 'hello';

for (var i = 0; i < s.length; i++) {
  console.log(s[i]);
}

console.log(i); // 5
```

上面程式碼中，變數`i`只用來控制迴圈，但是迴圈結束後，它並沒有消失，洩露成了全域性變數。

### ES6 的塊級作用域

`let`實際上為 JavaScript 新增了塊級作用域。

```javascript
function f1() {
  let n = 5;
  if (true) {
    let n = 10;
  }
  console.log(n); // 5
}
```

上面的函式有兩個程式碼塊，都聲明瞭變數`n`，執行後輸出 5。這表示外層程式碼塊不受內層程式碼塊的影響。如果兩次都使用`var`定義變數`n`，最後輸出的值才是 10。

ES6 允許塊級作用域的任意巢狀。

```javascript
{{{{
  {let insane = 'Hello World'}
  console.log(insane); // 報錯
}}}};
```

上面程式碼使用了一個五層的塊級作用域，每一層都是一個單獨的作用域。第四層作用域無法讀取第五層作用域的內部變數。

內層作用域可以定義外層作用域的同名變數。

```javascript
{{{{
  let insane = 'Hello World';
  {let insane = 'Hello World'}
}}}};
```

塊級作用域的出現，實際上使得獲得廣泛應用的匿名立即執行函式表示式（匿名 IIFE）不再必要了。

```javascript
// IIFE 寫法
(function () {
  var tmp = ...;
  ...
}());

// 塊級作用域寫法
{
  let tmp = ...;
  ...
}
```

### 塊級作用域與函式宣告

函式能不能在塊級作用域之中宣告？這是一個相當令人混淆的問題。

ES5 規定，函式只能在頂層作用域和函式作用域之中宣告，不能在塊級作用域宣告。

```javascript
// 情況一
if (true) {
  function f() {}
}

// 情況二
try {
  function f() {}
} catch(e) {
  // ...
}
```

上面兩種函式宣告，根據 ES5 的規定都是非法的。

但是，瀏覽器沒有遵守這個規定，為了相容以前的舊程式碼，還是支援在塊級作用域之中宣告函式，因此上面兩種情況實際都能執行，不會報錯。

ES6 引入了塊級作用域，明確允許在塊級作用域之中宣告函式。ES6 規定，塊級作用域之中，函式宣告語句的行為類似於`let`，在塊級作用域之外不可引用。

```javascript
function f() { console.log('I am outside!'); }

(function () {
  if (false) {
    // 重複宣告一次函式f
    function f() { console.log('I am inside!'); }
  }

  f();
}());
```

上面程式碼在 ES5 中執行，會得到“I am inside!”，因為在`if`內宣告的函式`f`會被提升到函式頭部，實際執行的程式碼如下。

```javascript
// ES5 環境
function f() { console.log('I am outside!'); }

(function () {
  function f() { console.log('I am inside!'); }
  if (false) {
  }
  f();
}());
```

ES6 就完全不一樣了，理論上會得到“I am outside!”。因為塊級作用域內宣告的函式類似於`let`，對作用域之外沒有影響。但是，如果你真的在 ES6 瀏覽器中執行一下上面的程式碼，是會報錯的，這是為什麼呢？

```javascript
// 瀏覽器的 ES6 環境
function f() { console.log('I am outside!'); }

(function () {
  if (false) {
    // 重複宣告一次函式f
    function f() { console.log('I am inside!'); }
  }

  f();
}());
// Uncaught TypeError: f is not a function
```

上面的程式碼在 ES6 瀏覽器中，都會報錯。

原來，如果改變了塊級作用域內宣告的函式的處理規則，顯然會對老程式碼產生很大影響。為了減輕因此產生的不相容問題，ES6 在[附錄 B](http://www.ecma-international.org/ecma-262/6.0/index.html#sec-block-level-function-declarations-web-legacy-compatibility-semantics)裡面規定，瀏覽器的實現可以不遵守上面的規定，有自己的[行為方式](http://stackoverflow.com/questions/31419897/what-are-the-precise-semantics-of-block-level-functions-in-es6)。

- 允許在塊級作用域內宣告函式。
- 函式宣告類似於`var`，即會提升到全域性作用域或函式作用域的頭部。
- 同時，函式宣告還會提升到所在的塊級作用域的頭部。

注意，上面三條規則只對 ES6 的瀏覽器實現有效，其他環境的實現不用遵守，還是將塊級作用域的函式聲明當作`let`處理。

根據這三條規則，瀏覽器的 ES6 環境中，塊級作用域內宣告的函式，行為類似於`var`宣告的變數。上面的例子實際執行的程式碼如下。

```javascript
// 瀏覽器的 ES6 環境
function f() { console.log('I am outside!'); }
(function () {
  var f = undefined;
  if (false) {
    function f() { console.log('I am inside!'); }
  }

  f();
}());
// Uncaught TypeError: f is not a function
```

考慮到環境導致的行為差異太大，應該避免在塊級作用域內宣告函式。如果確實需要，也應該寫成函式表示式，而不是函式宣告語句。

```javascript
// 塊級作用域內部的函式宣告語句，建議不要使用
{
  let a = 'secret';
  function f() {
    return a;
  }
}

// 塊級作用域內部，優先使用函式表示式
{
  let a = 'secret';
  let f = function () {
    return a;
  };
}
```

另外，還有一個需要注意的地方。ES6 的塊級作用域必須有大括號，如果沒有大括號，JavaScript 引擎就認為不存在塊級作用域。

```javascript
// 第一種寫法，報錯
if (true) let x = 1;

// 第二種寫法，不報錯
if (true) {
  let x = 1;
}
```

上面程式碼中，第一種寫法沒有大括號，所以不存在塊級作用域，而`let`只能出現在當前作用域的頂層，所以報錯。第二種寫法有大括號，所以塊級作用域成立。

函式宣告也是如此，嚴格模式下，函式只能宣告在當前作用域的頂層。

```javascript
// 不報錯
'use strict';
if (true) {
  function f() {}
}

// 報錯
'use strict';
if (true)
  function f() {}
```

## const 命令

### 基本用法

`const`宣告一個只讀的常量。一旦宣告，常量的值就不能改變。

```javascript
const PI = 3.1415;
PI // 3.1415

PI = 3;
// TypeError: Assignment to constant variable.
```

上面程式碼表明改變常量的值會報錯。

`const`宣告的變數不得改變值，這意味著，`const`一旦宣告變數，就必須立即初始化，不能留到以後賦值。

```javascript
const foo;
// SyntaxError: Missing initializer in const declaration
```

上面程式碼表示，對於`const`來說，只宣告不賦值，就會報錯。

`const`的作用域與`let`命令相同：只在宣告所在的塊級作用域內有效。

```javascript
if (true) {
  const MAX = 5;
}

MAX // Uncaught ReferenceError: MAX is not defined
```

`const`命令宣告的常量也是不提升，同樣存在暫時性死區，只能在宣告的位置後面使用。

```javascript
if (true) {
  console.log(MAX); // ReferenceError
  const MAX = 5;
}
```

上面程式碼在常量`MAX`宣告之前就呼叫，結果報錯。

`const`宣告的常量，也與`let`一樣不可重複宣告。

```javascript
var message = "Hello!";
let age = 25;

// 以下兩行都會報錯
const message = "Goodbye!";
const age = 30;
```

### 本質

`const`實際上保證的，並不是變數的值不得改動，而是變數指向的那個記憶體地址所儲存的資料不得改動。對於簡單型別的資料（數值、字串、布林值），值就儲存在變數指向的那個記憶體地址，因此等同於常量。但對於複合型別的資料（主要是物件和陣列），變數指向的記憶體地址，儲存的只是一個指向實際資料的指標，`const`只能保證這個指標是固定的（即總是指向另一個固定的地址），至於它指向的資料結構是不是可變的，就完全不能控制了。因此，將一個物件宣告為常量必須非常小心。

```javascript
const foo = {};

// 為 foo 新增一個屬性，可以成功
foo.prop = 123;
foo.prop // 123

// 將 foo 指向另一個物件，就會報錯
foo = {}; // TypeError: "foo" is read-only
```

上面程式碼中，常量`foo`儲存的是一個地址，這個地址指向一個物件。不可變的只是這個地址，即不能把`foo`指向另一個地址，但物件本身是可變的，所以依然可以為其新增新屬性。

下面是另一個例子。

```javascript
const a = [];
a.push('Hello'); // 可執行
a.length = 0;    // 可執行
a = ['Dave'];    // 報錯
```

上面程式碼中，常量`a`是一個數組，這個陣列本身是可寫的，但是如果將另一個數組賦值給`a`，就會報錯。

如果真的想將物件凍結，應該使用`Object.freeze`方法。

```javascript
const foo = Object.freeze({});

// 常規模式時，下面一行不起作用；
// 嚴格模式時，該行會報錯
foo.prop = 123;
```

上面程式碼中，常量`foo`指向一個凍結的物件，所以新增新屬性不起作用，嚴格模式時還會報錯。

除了將物件本身凍結，物件的屬性也應該凍結。下面是一個將物件徹底凍結的函式。

```javascript
var constantize = (obj) => {
  Object.freeze(obj);
  Object.keys(obj).forEach( (key, i) => {
    if ( typeof obj[key] === 'object' ) {
      constantize( obj[key] );
    }
  });
};
```

### ES6 宣告變數的六種方法

ES5 只有兩種宣告變數的方法：`var`命令和`function`命令。ES6 除了新增`let`和`const`命令，後面章節還會提到，另外兩種宣告變數的方法：`import`命令和`class`命令。所以，ES6 一共有 6 種宣告變數的方法。

## 頂層物件的屬性

頂層物件，在瀏覽器環境指的是`window`物件，在 Node 指的是`global`物件。ES5 之中，頂層物件的屬性與全域性變數是等價的。

```javascript
window.a = 1;
a // 1

a = 2;
window.a // 2
```

上面程式碼中，頂層物件的屬性賦值與全域性變數的賦值，是同一件事。

頂層物件的屬性與全域性變數掛鉤，被認為是 JavaScript 語言最大的設計敗筆之一。這樣的設計帶來了幾個很大的問題，首先是沒法在編譯時就報出變數未宣告的錯誤，只有執行時才能知道（因為全域性變數可能是頂層物件的屬性創造的，而屬性的創造是動態的）；其次，程式設計師很容易不知不覺地就建立了全域性變數（比如打字出錯）；最後，頂層物件的屬性是到處可以讀寫的，這非常不利於模組化程式設計。另一方面，`window`物件有實體含義，指的是瀏覽器的視窗物件，頂層物件是一個有實體含義的物件，也是不合適的。

ES6 為了改變這一點，一方面規定，為了保持相容性，`var`命令和`function`命令宣告的全域性變數，依舊是頂層物件的屬性；另一方面規定，`let`命令、`const`命令、`class`命令宣告的全域性變數，不屬於頂層物件的屬性。也就是說，從 ES6 開始，全域性變數將逐步與頂層物件的屬性脫鉤。

```javascript
var a = 1;
// 如果在 Node 的 REPL 環境，可以寫成 global.a
// 或者採用通用方法，寫成 this.a
window.a // 1

let b = 1;
window.b // undefined
```

上面程式碼中，全域性變數`a`由`var`命令宣告，所以它是頂層物件的屬性；全域性變數`b`由`let`命令宣告，所以它不是頂層物件的屬性，返回`undefined`。

## globalThis 物件

JavaScript 語言存在一個頂層物件，它提供全域性環境（即全域性作用域），所有程式碼都是在這個環境中執行。但是，頂層物件在各種實現裡面是不統一的。

- 瀏覽器裡面，頂層物件是`window`，但 Node 和 Web Worker 沒有`window`。
- 瀏覽器和 Web Worker 裡面，`self`也指向頂層物件，但是 Node 沒有`self`。
- Node 裡面，頂層物件是`global`，但其他環境都不支援。

同一段程式碼為了能夠在各種環境，都能取到頂層物件，現在一般是使用`this`變數，但是有侷限性。

- 全域性環境中，`this`會返回頂層物件。但是，Node.js 模組中`this`返回的是當前模組，ES6 模組中`this`返回的是`undefined`。
- 函式裡面的`this`，如果函式不是作為物件的方法執行，而是單純作為函式執行，`this`會指向頂層物件。但是，嚴格模式下，這時`this`會返回`undefined`。
- 不管是嚴格模式，還是普通模式，`new Function('return this')()`，總是會返回全域性物件。但是，如果瀏覽器用了 CSP（Content Security Policy，內容安全策略），那麼`eval`、`new Function`這些方法都可能無法使用。

綜上所述，很難找到一種方法，可以在所有情況下，都取到頂層物件。下面是兩種勉強可以使用的方法。

```javascript
// 方法一
(typeof window !== 'undefined'
   ? window
   : (typeof process === 'object' &&
      typeof require === 'function' &&
      typeof global === 'object')
     ? global
     : this);

// 方法二
var getGlobal = function () {
  if (typeof self !== 'undefined') { return self; }
  if (typeof window !== 'undefined') { return window; }
  if (typeof global !== 'undefined') { return global; }
  throw new Error('unable to locate global object');
};
```

[ES2020](https://github.com/tc39/proposal-global) 在語言標準的層面，引入`globalThis`作為頂層物件。也就是說，任何環境下，`globalThis`都是存在的，都可以從它拿到頂層物件，指向全域性環境下的`this`。

墊片庫[`global-this`](https://github.com/ungap/global-this)模擬了這個提案，可以在所有環境拿到`globalThis`。

