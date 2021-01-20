# 數值的擴充套件

## 二進位制和八進位制表示法

ES6 提供了二進位制和八進位制數值的新的寫法，分別用字首`0b`（或`0B`）和`0o`（或`0O`）表示。

```javascript
0b111110111 === 503 // true
0o767 === 503 // true
```

從 ES5 開始，在嚴格模式之中，八進位制就不再允許使用字首`0`表示，ES6 進一步明確，要使用字首`0o`表示。

```javascript
// 非嚴格模式
(function(){
  console.log(0o11 === 011);
})() // true

// 嚴格模式
(function(){
  'use strict';
  console.log(0o11 === 011);
})() // Uncaught SyntaxError: Octal literals are not allowed in strict mode.
```

如果要將`0b`和`0o`字首的字串數值轉為十進位制，要使用`Number`方法。

```javascript
Number('0b111')  // 7
Number('0o10')  // 8
```

## Number.isFinite(), Number.isNaN()

ES6 在`Number`物件上，新提供了`Number.isFinite()`和`Number.isNaN()`兩個方法。

`Number.isFinite()`用來檢查一個數值是否為有限的（finite），即不是`Infinity`。

```javascript
Number.isFinite(15); // true
Number.isFinite(0.8); // true
Number.isFinite(NaN); // false
Number.isFinite(Infinity); // false
Number.isFinite(-Infinity); // false
Number.isFinite('foo'); // false
Number.isFinite('15'); // false
Number.isFinite(true); // false
```

注意，如果引數型別不是數值，`Number.isFinite`一律返回`false`。

`Number.isNaN()`用來檢查一個值是否為`NaN`。

```javascript
Number.isNaN(NaN) // true
Number.isNaN(15) // false
Number.isNaN('15') // false
Number.isNaN(true) // false
Number.isNaN(9/NaN) // true
Number.isNaN('true' / 0) // true
Number.isNaN('true' / 'true') // true
```

如果引數型別不是`NaN`，`Number.isNaN`一律返回`false`。

它們與傳統的全域性方法`isFinite()`和`isNaN()`的區別在於，傳統方法先呼叫`Number()`將非數值的值轉為數值，再進行判斷，而這兩個新方法只對數值有效，`Number.isFinite()`對於非數值一律返回`false`, `Number.isNaN()`只有對於`NaN`才返回`true`，非`NaN`一律返回`false`。

```javascript
isFinite(25) // true
isFinite("25") // true
Number.isFinite(25) // true
Number.isFinite("25") // false

isNaN(NaN) // true
isNaN("NaN") // true
Number.isNaN(NaN) // true
Number.isNaN("NaN") // false
Number.isNaN(1) // false
```

## Number.parseInt(), Number.parseFloat()

ES6 將全域性方法`parseInt()`和`parseFloat()`，移植到`Number`物件上面，行為完全保持不變。

```javascript
// ES5的寫法
parseInt('12.34') // 12
parseFloat('123.45#') // 123.45

// ES6的寫法
Number.parseInt('12.34') // 12
Number.parseFloat('123.45#') // 123.45
```

這樣做的目的，是逐步減少全域性性方法，使得語言逐步模組化。

```javascript
Number.parseInt === parseInt // true
Number.parseFloat === parseFloat // true
```

## Number.isInteger()

`Number.isInteger()`用來判斷一個數值是否為整數。

```javascript
Number.isInteger(25) // true
Number.isInteger(25.1) // false
```

JavaScript 內部，整數和浮點數採用的是同樣的儲存方法，所以 25  和 25.0 被視為同一個值。

```javascript
Number.isInteger(25) // true
Number.isInteger(25.0) // true
```

如果引數不是數值，`Number.isInteger`返回`false`。

```javascript
Number.isInteger() // false
Number.isInteger(null) // false
Number.isInteger('15') // false
Number.isInteger(true) // false
```

注意，由於 JavaScript 採用 IEEE 754 標準，數值儲存為64位雙精度格式，數值精度最多可以達到 53 個二進位制位（1 個隱藏位與 52 個有效位）。如果數值的精度超過這個限度，第54位及後面的位就會被丟棄，這種情況下，`Number.isInteger`可能會誤判。

```javascript
Number.isInteger(3.0000000000000002) // true
```

上面程式碼中，`Number.isInteger`的引數明明不是整數，但是會返回`true`。原因就是這個小數的精度達到了小數點後16個十進位制位，轉成二進位制位超過了53個二進位制位，導致最後的那個`2`被丟棄了。

類似的情況還有，如果一個數值的絕對值小於`Number.MIN_VALUE`（5E-324），即小於 JavaScript 能夠分辨的最小值，會被自動轉為 0。這時，`Number.isInteger`也會誤判。

```javascript
Number.isInteger(5E-324) // false
Number.isInteger(5E-325) // true
```

上面程式碼中，`5E-325`由於值太小，會被自動轉為0，因此返回`true`。

總之，如果對資料精度的要求較高，不建議使用`Number.isInteger()`判斷一個數值是否為整數。

## Number.EPSILON

ES6 在`Number`物件上面，新增一個極小的常量`Number.EPSILON`。根據規格，它表示 1 與大於 1 的最小浮點數之間的差。

對於 64 位浮點數來說，大於 1 的最小浮點數相當於二進位制的`1.00..001`，小數點後面有連續 51 個零。這個值減去 1 之後，就等於 2 的 -52 次方。

```javascript
Number.EPSILON === Math.pow(2, -52)
// true
Number.EPSILON
// 2.220446049250313e-16
Number.EPSILON.toFixed(20)
// "0.00000000000000022204"
```

`Number.EPSILON`實際上是 JavaScript 能夠表示的最小精度。誤差如果小於這個值，就可以認為已經沒有意義了，即不存在誤差了。

引入一個這麼小的量的目的，在於為浮點數計算，設定一個誤差範圍。我們知道浮點數計算是不精確的。

```javascript
0.1 + 0.2
// 0.30000000000000004

0.1 + 0.2 - 0.3
// 5.551115123125783e-17

5.551115123125783e-17.toFixed(20)
// '0.00000000000000005551'
```

上面程式碼解釋了，為什麼比較`0.1 + 0.2`與`0.3`得到的結果是`false`。

```javascript
0.1 + 0.2 === 0.3 // false
```

`Number.EPSILON`可以用來設定“能夠接受的誤差範圍”。比如，誤差範圍設為 2 的-50 次方（即`Number.EPSILON * Math.pow(2, 2)`），即如果兩個浮點數的差小於這個值，我們就認為這兩個浮點數相等。

```javascript
5.551115123125783e-17 < Number.EPSILON * Math.pow(2, 2)
// true
```

因此，`Number.EPSILON`的實質是一個可以接受的最小誤差範圍。

```javascript
function withinErrorMargin (left, right) {
  return Math.abs(left - right) < Number.EPSILON * Math.pow(2, 2);
}

0.1 + 0.2 === 0.3 // false
withinErrorMargin(0.1 + 0.2, 0.3) // true

1.1 + 1.3 === 2.4 // false
withinErrorMargin(1.1 + 1.3, 2.4) // true
```

上面的程式碼為浮點數運算，部署了一個誤差檢查函式。

## 安全整數和 Number.isSafeInteger()

JavaScript 能夠準確表示的整數範圍在`-2^53`到`2^53`之間（不含兩個端點），超過這個範圍，無法精確表示這個值。

```javascript
Math.pow(2, 53) // 9007199254740992

9007199254740992  // 9007199254740992
9007199254740993  // 9007199254740992

Math.pow(2, 53) === Math.pow(2, 53) + 1
// true
```

上面程式碼中，超出 2 的 53 次方之後，一個數就不精確了。

ES6 引入了`Number.MAX_SAFE_INTEGER`和`Number.MIN_SAFE_INTEGER`這兩個常量，用來表示這個範圍的上下限。

```javascript
Number.MAX_SAFE_INTEGER === Math.pow(2, 53) - 1
// true
Number.MAX_SAFE_INTEGER === 9007199254740991
// true

Number.MIN_SAFE_INTEGER === -Number.MAX_SAFE_INTEGER
// true
Number.MIN_SAFE_INTEGER === -9007199254740991
// true
```

上面程式碼中，可以看到 JavaScript 能夠精確表示的極限。

`Number.isSafeInteger()`則是用來判斷一個整數是否落在這個範圍之內。

```javascript
Number.isSafeInteger('a') // false
Number.isSafeInteger(null) // false
Number.isSafeInteger(NaN) // false
Number.isSafeInteger(Infinity) // false
Number.isSafeInteger(-Infinity) // false

Number.isSafeInteger(3) // true
Number.isSafeInteger(1.2) // false
Number.isSafeInteger(9007199254740990) // true
Number.isSafeInteger(9007199254740992) // false

Number.isSafeInteger(Number.MIN_SAFE_INTEGER - 1) // false
Number.isSafeInteger(Number.MIN_SAFE_INTEGER) // true
Number.isSafeInteger(Number.MAX_SAFE_INTEGER) // true
Number.isSafeInteger(Number.MAX_SAFE_INTEGER + 1) // false
```

這個函式的實現很簡單，就是跟安全整數的兩個邊界值比較一下。

```javascript
Number.isSafeInteger = function (n) {
  return (typeof n === 'number' &&
    Math.round(n) === n &&
    Number.MIN_SAFE_INTEGER <= n &&
    n <= Number.MAX_SAFE_INTEGER);
}
```

實際使用這個函式時，需要注意。驗證運算結果是否落在安全整數的範圍內，不要只驗證運算結果，而要同時驗證參與運算的每個值。

```javascript
Number.isSafeInteger(9007199254740993)
// false
Number.isSafeInteger(990)
// true
Number.isSafeInteger(9007199254740993 - 990)
// true
9007199254740993 - 990
// 返回結果 9007199254740002
// 正確答案應該是 9007199254740003
```

上面程式碼中，`9007199254740993`不是一個安全整數，但是`Number.isSafeInteger`會返回結果，顯示計算結果是安全的。這是因為，這個數超出了精度範圍，導致在計算機內部，以`9007199254740992`的形式儲存。

```javascript
9007199254740993 === 9007199254740992
// true
```

所以，如果只驗證運算結果是否為安全整數，很可能得到錯誤結果。下面的函式可以同時驗證兩個運算數和運算結果。

```javascript
function trusty (left, right, result) {
  if (
    Number.isSafeInteger(left) &&
    Number.isSafeInteger(right) &&
    Number.isSafeInteger(result)
  ) {
    return result;
  }
  throw new RangeError('Operation cannot be trusted!');
}

trusty(9007199254740993, 990, 9007199254740993 - 990)
// RangeError: Operation cannot be trusted!

trusty(1, 2, 3)
// 3
```

## Math 物件的擴充套件

ES6 在 Math 物件上新增了 17 個與數學相關的方法。所有這些方法都是靜態方法，只能在 Math 物件上呼叫。

### Math.trunc()

`Math.trunc`方法用於去除一個數的小數部分，返回整數部分。

```javascript
Math.trunc(4.1) // 4
Math.trunc(4.9) // 4
Math.trunc(-4.1) // -4
Math.trunc(-4.9) // -4
Math.trunc(-0.1234) // -0
```

對於非數值，`Math.trunc`內部使用`Number`方法將其先轉為數值。

```javascript
Math.trunc('123.456') // 123
Math.trunc(true) //1
Math.trunc(false) // 0
Math.trunc(null) // 0
```

對於空值和無法擷取整數的值，返回`NaN`。

```javascript
Math.trunc(NaN);      // NaN
Math.trunc('foo');    // NaN
Math.trunc();         // NaN
Math.trunc(undefined) // NaN
```

對於沒有部署這個方法的環境，可以用下面的程式碼模擬。

```javascript
Math.trunc = Math.trunc || function(x) {
  return x < 0 ? Math.ceil(x) : Math.floor(x);
};
```

### Math.sign()

`Math.sign`方法用來判斷一個數到底是正數、負數、還是零。對於非數值，會先將其轉換為數值。

它會返回五種值。

- 引數為正數，返回`+1`；
- 引數為負數，返回`-1`；
- 引數為 0，返回`0`；
- 引數為-0，返回`-0`;
- 其他值，返回`NaN`。

```javascript
Math.sign(-5) // -1
Math.sign(5) // +1
Math.sign(0) // +0
Math.sign(-0) // -0
Math.sign(NaN) // NaN
```

如果引數是非數值，會自動轉為數值。對於那些無法轉為數值的值，會返回`NaN`。

```javascript
Math.sign('')  // 0
Math.sign(true)  // +1
Math.sign(false)  // 0
Math.sign(null)  // 0
Math.sign('9')  // +1
Math.sign('foo')  // NaN
Math.sign()  // NaN
Math.sign(undefined)  // NaN
```

對於沒有部署這個方法的環境，可以用下面的程式碼模擬。

```javascript
Math.sign = Math.sign || function(x) {
  x = +x; // convert to a number
  if (x === 0 || isNaN(x)) {
    return x;
  }
  return x > 0 ? 1 : -1;
};
```

### Math.cbrt()

`Math.cbrt()`方法用於計算一個數的立方根。

```javascript
Math.cbrt(-1) // -1
Math.cbrt(0)  // 0
Math.cbrt(1)  // 1
Math.cbrt(2)  // 1.2599210498948732
```

對於非數值，`Math.cbrt()`方法內部也是先使用`Number()`方法將其轉為數值。

```javascript
Math.cbrt('8') // 2
Math.cbrt('hello') // NaN
```

對於沒有部署這個方法的環境，可以用下面的程式碼模擬。

```javascript
Math.cbrt = Math.cbrt || function(x) {
  var y = Math.pow(Math.abs(x), 1/3);
  return x < 0 ? -y : y;
};
```

### Math.clz32()

`Math.clz32()`方法將引數轉為 32 位無符號整數的形式，然後返回這個 32 位值裡面有多少個前導 0。

```javascript
Math.clz32(0) // 32
Math.clz32(1) // 31
Math.clz32(1000) // 22
Math.clz32(0b01000000000000000000000000000000) // 1
Math.clz32(0b00100000000000000000000000000000) // 2
```

上面程式碼中，0 的二進位制形式全為 0，所以有 32 個前導 0；1 的二進位制形式是`0b1`，只佔 1 位，所以 32 位之中有 31 個前導 0；1000 的二進位制形式是`0b1111101000`，一共有 10 位，所以 32 位之中有 22 個前導 0。

`clz32`這個函式名就來自”count leading zero bits in 32-bit binary representation of a number“（計算一個數的 32 位二進位制形式的前導 0 的個數）的縮寫。

左移運算子（`<<`）與`Math.clz32`方法直接相關。

```javascript
Math.clz32(0) // 32
Math.clz32(1) // 31
Math.clz32(1 << 1) // 30
Math.clz32(1 << 2) // 29
Math.clz32(1 << 29) // 2
```

對於小數，`Math.clz32`方法只考慮整數部分。

```javascript
Math.clz32(3.2) // 30
Math.clz32(3.9) // 30
```

對於空值或其他型別的值，`Math.clz32`方法會將它們先轉為數值，然後再計算。

```javascript
Math.clz32() // 32
Math.clz32(NaN) // 32
Math.clz32(Infinity) // 32
Math.clz32(null) // 32
Math.clz32('foo') // 32
Math.clz32([]) // 32
Math.clz32({}) // 32
Math.clz32(true) // 31
```

### Math.imul()

`Math.imul`方法返回兩個數以 32 位帶符號整數形式相乘的結果，返回的也是一個 32 位的帶符號整數。

```javascript
Math.imul(2, 4)   // 8
Math.imul(-1, 8)  // -8
Math.imul(-2, -2) // 4
```

如果只考慮最後 32 位，大多數情況下，`Math.imul(a, b)`與`a * b`的結果是相同的，即該方法等同於`(a * b)|0`的效果（超過 32 位的部分溢位）。之所以需要部署這個方法，是因為 JavaScript 有精度限制，超過 2 的 53 次方的值無法精確表示。這就是說，對於那些很大的數的乘法，低位數值往往都是不精確的，`Math.imul`方法可以返回正確的低位數值。

```javascript
(0x7fffffff * 0x7fffffff)|0 // 0
```

上面這個乘法算式，返回結果為 0。但是由於這兩個二進位制數的最低位都是 1，所以這個結果肯定是不正確的，因為根據二進位制乘法，計算結果的二進位制最低位應該也是 1。這個錯誤就是因為它們的乘積超過了 2 的 53 次方，JavaScript 無法儲存額外的精度，就把低位的值都變成了 0。`Math.imul`方法可以返回正確的值 1。

```javascript
Math.imul(0x7fffffff, 0x7fffffff) // 1
```

### Math.fround()

`Math.fround`方法返回一個數的32位單精度浮點數形式。

對於32位單精度格式來說，數值精度是24個二進位制位（1 位隱藏位與 23 位有效位），所以對於 -2<sup>24</sup> 至 2<sup>24</sup> 之間的整數（不含兩個端點），返回結果與引數本身一致。

```javascript
Math.fround(0)   // 0
Math.fround(1)   // 1
Math.fround(2 ** 24 - 1)   // 16777215
```

如果引數的絕對值大於 2<sup>24</sup>，返回的結果便開始丟失精度。

```javascript
Math.fround(2 ** 24)       // 16777216
Math.fround(2 ** 24 + 1)   // 16777216
```

`Math.fround`方法的主要作用，是將64位雙精度浮點數轉為32位單精度浮點數。如果小數的精度超過24個二進位制位，返回值就會不同於原值，否則返回值不變（即與64位雙精度值一致）。

```javascript
// 未丟失有效精度
Math.fround(1.125) // 1.125
Math.fround(7.25)  // 7.25

// 丟失精度
Math.fround(0.3)   // 0.30000001192092896
Math.fround(0.7)   // 0.699999988079071
Math.fround(1.0000000123) // 1
```

對於 `NaN` 和 `Infinity`，此方法返回原值。對於其它型別的非數值，`Math.fround` 方法會先將其轉為數值，再返回單精度浮點數。

```javascript
Math.fround(NaN)      // NaN
Math.fround(Infinity) // Infinity

Math.fround('5')      // 5
Math.fround(true)     // 1
Math.fround(null)     // 0
Math.fround([])       // 0
Math.fround({})       // NaN
```

對於沒有部署這個方法的環境，可以用下面的程式碼模擬。

```javascript
Math.fround = Math.fround || function (x) {
  return new Float32Array([x])[0];
};
```

### Math.hypot()

`Math.hypot`方法返回所有引數的平方和的平方根。

```javascript
Math.hypot(3, 4);        // 5
Math.hypot(3, 4, 5);     // 7.0710678118654755
Math.hypot();            // 0
Math.hypot(NaN);         // NaN
Math.hypot(3, 4, 'foo'); // NaN
Math.hypot(3, 4, '5');   // 7.0710678118654755
Math.hypot(-3);          // 3
```

上面程式碼中，3 的平方加上 4 的平方，等於 5 的平方。

如果引數不是數值，`Math.hypot`方法會將其轉為數值。只要有一個引數無法轉為數值，就會返回 NaN。

### 對數方法

ES6 新增了 4 個對數相關方法。

**（1） Math.expm1()**

`Math.expm1(x)`返回 e<sup>x</sup> - 1，即`Math.exp(x) - 1`。

```javascript
Math.expm1(-1) // -0.6321205588285577
Math.expm1(0)  // 0
Math.expm1(1)  // 1.718281828459045
```

對於沒有部署這個方法的環境，可以用下面的程式碼模擬。

```javascript
Math.expm1 = Math.expm1 || function(x) {
  return Math.exp(x) - 1;
};
```

**（2）Math.log1p()**

`Math.log1p(x)`方法返回`1 + x`的自然對數，即`Math.log(1 + x)`。如果`x`小於-1，返回`NaN`。

```javascript
Math.log1p(1)  // 0.6931471805599453
Math.log1p(0)  // 0
Math.log1p(-1) // -Infinity
Math.log1p(-2) // NaN
```

對於沒有部署這個方法的環境，可以用下面的程式碼模擬。

```javascript
Math.log1p = Math.log1p || function(x) {
  return Math.log(1 + x);
};
```

**（3）Math.log10()**

`Math.log10(x)`返回以 10 為底的`x`的對數。如果`x`小於 0，則返回 NaN。

```javascript
Math.log10(2)      // 0.3010299956639812
Math.log10(1)      // 0
Math.log10(0)      // -Infinity
Math.log10(-2)     // NaN
Math.log10(100000) // 5
```

對於沒有部署這個方法的環境，可以用下面的程式碼模擬。

```javascript
Math.log10 = Math.log10 || function(x) {
  return Math.log(x) / Math.LN10;
};
```

**（4）Math.log2()**

`Math.log2(x)`返回以 2 為底的`x`的對數。如果`x`小於 0，則返回 NaN。

```javascript
Math.log2(3)       // 1.584962500721156
Math.log2(2)       // 1
Math.log2(1)       // 0
Math.log2(0)       // -Infinity
Math.log2(-2)      // NaN
Math.log2(1024)    // 10
Math.log2(1 << 29) // 29
```

對於沒有部署這個方法的環境，可以用下面的程式碼模擬。

```javascript
Math.log2 = Math.log2 || function(x) {
  return Math.log(x) / Math.LN2;
};
```

### 雙曲函式方法

ES6 新增了 6 個雙曲函式方法。

- `Math.sinh(x)` 返回`x`的雙曲正弦（hyperbolic sine）
- `Math.cosh(x)` 返回`x`的雙曲餘弦（hyperbolic cosine）
- `Math.tanh(x)` 返回`x`的雙曲正切（hyperbolic tangent）
- `Math.asinh(x)` 返回`x`的反雙曲正弦（inverse hyperbolic sine）
- `Math.acosh(x)` 返回`x`的反雙曲餘弦（inverse hyperbolic cosine）
- `Math.atanh(x)` 返回`x`的反雙曲正切（inverse hyperbolic tangent）

## 指數運算子

ES2016 新增了一個指數運算子（`**`）。

```javascript
2 ** 2 // 4
2 ** 3 // 8
```

這個運算子的一個特點是右結合，而不是常見的左結合。多個指數運算子連用時，是從最右邊開始計算的。

```javascript
// 相當於 2 ** (3 ** 2)
2 ** 3 ** 2
// 512
```

上面程式碼中，首先計算的是第二個指數運算子，而不是第一個。

指數運算子可以與等號結合，形成一個新的賦值運算子（`**=`）。

```javascript
let a = 1.5;
a **= 2;
// 等同於 a = a * a;

let b = 4;
b **= 3;
// 等同於 b = b * b * b;
```

## BigInt 資料型別

### 簡介

JavaScript 所有數字都儲存成 64 位浮點數，這給數值的表示帶來了兩大限制。一是數值的精度只能到 53 個二進位制位（相當於 16 個十進位制位），大於這個範圍的整數，JavaScript 是無法精確表示的，這使得 JavaScript 不適合進行科學和金融方面的精確計算。二是大於或等於2的1024次方的數值，JavaScript 無法表示，會返回`Infinity`。

```javascript
// 超過 53 個二進位制位的數值，無法保持精度
Math.pow(2, 53) === Math.pow(2, 53) + 1 // true

// 超過 2 的 1024 次方的數值，無法表示
Math.pow(2, 1024) // Infinity
```

[ES2020](https://github.com/tc39/proposal-bigint) 引入了一種新的資料型別 BigInt（大整數），來解決這個問題，這是 ECMAScript 的第八種資料型別。BigInt 只用來表示整數，沒有位數的限制，任何位數的整數都可以精確表示。

```javascript
const a = 2172141653n;
const b = 15346349309n;

// BigInt 可以保持精度
a * b // 33334444555566667777n

// 普通整數無法保持精度
Number(a) * Number(b) // 33334444555566670000
```

為了與 Number 型別區別，BigInt 型別的資料必須新增字尾`n`。

```javascript
1234 // 普通整數
1234n // BigInt

// BigInt 的運算
1n + 2n // 3n
```

BigInt 同樣可以使用各種進製表示，都要加上字尾`n`。

```javascript
0b1101n // 二進位制
0o777n // 八進位制
0xFFn // 十六進位制
```

BigInt 與普通整數是兩種值，它們之間並不相等。

```javascript
42n === 42 // false
```

`typeof`運算子對於 BigInt 型別的資料返回`bigint`。

```javascript
typeof 123n // 'bigint'
```

BigInt 可以使用負號（`-`），但是不能使用正號（`+`），因為會與 asm.js 衝突。

```javascript
-42n // 正確
+42n // 報錯
```

JavaScript 以前不能計算70的階乘（即`70!`），因為超出了可以表示的精度。

```javascript
let p = 1;
for (let i = 1; i <= 70; i++) {
  p *= i;
}
console.log(p); // 1.197857166996989e+100
```

現在支援大整數了，就可以算了，瀏覽器的開發者工具執行下面程式碼，就OK。

```javascript
let p = 1n;
for (let i = 1n; i <= 70n; i++) {
  p *= i;
}
console.log(p); // 11978571...00000000n
```

### BigInt 物件

JavaScript 原生提供`BigInt`物件，可以用作建構函式生成 BigInt 型別的數值。轉換規則基本與`Number()`一致，將其他型別的值轉為 BigInt。

```javascript
BigInt(123) // 123n
BigInt('123') // 123n
BigInt(false) // 0n
BigInt(true) // 1n
```

`BigInt()`建構函式必須有引數，而且引數必須可以正常轉為數值，下面的用法都會報錯。

```javascript
new BigInt() // TypeError
BigInt(undefined) //TypeError
BigInt(null) // TypeError
BigInt('123n') // SyntaxError
BigInt('abc') // SyntaxError
```

上面程式碼中，尤其值得注意字串`123n`無法解析成 Number 型別，所以會報錯。

引數如果是小數，也會報錯。

```javascript
BigInt(1.5) // RangeError
BigInt('1.5') // SyntaxError
```

BigInt 物件繼承了 Object 物件的兩個例項方法。

- `BigInt.prototype.toString()`
- `BigInt.prototype.valueOf()`

它還繼承了 Number 物件的一個例項方法。

- `BigInt.prototype.toLocaleString()`

此外，還提供了三個靜態方法。

- `BigInt.asUintN(width, BigInt)`： 給定的 BigInt 轉為 0 到 2<sup>width</sup> - 1 之間對應的值。
- `BigInt.asIntN(width, BigInt)`：給定的 BigInt 轉為 -2<sup>width - 1</sup> 到 2<sup>width - 1</sup> - 1 之間對應的值。
- `BigInt.parseInt(string[, radix])`：近似於`Number.parseInt()`，將一個字串轉換成指定進位制的 BigInt。

```javascript
const max = 2n ** (64n - 1n) - 1n;

BigInt.asIntN(64, max)
// 9223372036854775807n
BigInt.asIntN(64, max + 1n)
// -9223372036854775808n
BigInt.asUintN(64, max + 1n)
// 9223372036854775808n
```

上面程式碼中，`max`是64位帶符號的 BigInt 所能表示的最大值。如果對這個值加`1n`，`BigInt.asIntN()`將會返回一個負值，因為這時新增的一位將被解釋為符號位。而`BigInt.asUintN()`方法由於不存在符號位，所以可以正確返回結果。

如果`BigInt.asIntN()`和`BigInt.asUintN()`指定的位數，小於數值本身的位數，那麼頭部的位將被捨棄。

```javascript
const max = 2n ** (64n - 1n) - 1n;

BigInt.asIntN(32, max) // -1n
BigInt.asUintN(32, max) // 4294967295n
```

上面程式碼中，`max`是一個64位的 BigInt，如果轉為32位，前面的32位都會被捨棄。

下面是`BigInt.parseInt()`的例子。

```javascript
// Number.parseInt() 與 BigInt.parseInt() 的對比
Number.parseInt('9007199254740993', 10)
// 9007199254740992
BigInt.parseInt('9007199254740993', 10)
// 9007199254740993n
```

上面程式碼中，由於有效數字超出了最大限度，`Number.parseInt`方法返回的結果是不精確的，而`BigInt.parseInt`方法正確返回了對應的 BigInt。

對於二進位制陣列，BigInt 新增了兩個型別`BigUint64Array`和`BigInt64Array`，這兩種資料型別返回的都是64位 BigInt。`DataView`物件的例項方法`DataView.prototype.getBigInt64()`和`DataView.prototype.getBigUint64()`，返回的也是 BigInt。

### 轉換規則

可以使用`Boolean()`、`Number()`和`String()`這三個方法，將 BigInt 可以轉為布林值、數值和字串型別。

```javascript
Boolean(0n) // false
Boolean(1n) // true
Number(1n)  // 1
String(1n)  // "1"
```

上面程式碼中，注意最後一個例子，轉為字串時後綴`n`會消失。

另外，取反運算子（`!`）也可以將 BigInt 轉為布林值。

```javascript
!0n // true
!1n // false
```

### 數學運算

數學運算方面，BigInt 型別的`+`、`-`、`*`和`**`這四個二元運算子，與 Number 型別的行為一致。除法運算`/`會捨去小數部分，返回一個整數。

```javascript
9n / 5n
// 1n
```

幾乎所有的數值運算子都可以用在 BigInt，但是有兩個例外。

- 不帶符號的右移位運算子`>>>`
- 一元的求正運算子`+`

上面兩個運算子用在 BigInt 會報錯。前者是因為`>>>`運算子是不帶符號的，但是 BigInt 總是帶有符號的，導致該運算無意義，完全等同於右移運算子`>>`。後者是因為一元運算子`+`在 asm.js 裡面總是返回 Number 型別，為了不破壞 asm.js 就規定`+1n`會報錯。

BigInt 不能與普通數值進行混合運算。

```javascript
1n + 1.3 // 報錯
```

上面程式碼報錯是因為無論返回的是 BigInt 或 Number，都會導致丟失精度資訊。比如`(2n**53n + 1n) + 0.5`這個表示式，如果返回 BigInt 型別，`0.5`這個小數部分會丟失；如果返回 Number 型別，有效精度只能保持 53 位，導致精度下降。

同樣的原因，如果一個標準庫函式的引數預期是 Number 型別，但是得到的是一個 BigInt，就會報錯。

```javascript
// 錯誤的寫法
Math.sqrt(4n) // 報錯

// 正確的寫法
Math.sqrt(Number(4n)) // 2
```

上面程式碼中，`Math.sqrt`的引數預期是 Number 型別，如果是 BigInt 就會報錯，必須先用`Number`方法轉一下型別，才能進行計算。

asm.js 裡面，`|0`跟在一個數值的後面會返回一個32位整數。根據不能與 Number 型別混合運算的規則，BigInt 如果與`|0`進行運算會報錯。

```javascript
1n | 0 // 報錯
```

### 其他運算

BigInt 對應的布林值，與 Number 型別一致，即`0n`會轉為`false`，其他值轉為`true`。

```javascript
if (0n) {
  console.log('if');
} else {
  console.log('else');
}
// else
```

上面程式碼中，`0n`對應`false`，所以會進入`else`子句。

比較運算子（比如`>`）和相等運算子（`==`）允許 BigInt 與其他型別的值混合計算，因為這樣做不會損失精度。

```javascript
0n < 1 // true
0n < true // true
0n == 0 // true
0n == false // true
0n === 0 // false
```

BigInt 與字串混合運算時，會先轉為字串，再進行運算。

```javascript
'' + 123n // "123"
```

