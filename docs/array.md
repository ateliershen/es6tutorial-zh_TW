# 陣列的擴充套件

## 擴充套件運算子

### 含義

擴充套件運算子（spread）是三個點（`...`）。它好比 rest 引數的逆運算，將一個數組轉為用逗號分隔的引數序列。

```javascript
console.log(...[1, 2, 3])
// 1 2 3

console.log(1, ...[2, 3, 4], 5)
// 1 2 3 4 5

[...document.querySelectorAll('div')]
// [<div>, <div>, <div>]
```

該運算子主要用於函式呼叫。

```javascript
function push(array, ...items) {
  array.push(...items);
}

function add(x, y) {
  return x + y;
}

const numbers = [4, 38];
add(...numbers) // 42
```

上面程式碼中，`array.push(...items)`和`add(...numbers)`這兩行，都是函式的呼叫，它們都使用了擴充套件運算子。該運算子將一個數組，變為引數序列。

擴充套件運算子與正常的函式引數可以結合使用，非常靈活。

```javascript
function f(v, w, x, y, z) { }
const args = [0, 1];
f(-1, ...args, 2, ...[3]);
```

擴充套件運算子後面還可以放置表示式。

```javascript
const arr = [
  ...(x > 0 ? ['a'] : []),
  'b',
];
```

如果擴充套件運算子後面是一個空陣列，則不產生任何效果。

```javascript
[...[], 1]
// [1]
```

注意，只有函式呼叫時，擴充套件運算子才可以放在圓括號中，否則會報錯。

```javascript
(...[1, 2])
// Uncaught SyntaxError: Unexpected number

console.log((...[1, 2]))
// Uncaught SyntaxError: Unexpected number

console.log(...[1, 2])
// 1 2
```

上面三種情況，擴充套件運算子都放在圓括號裡面，但是前兩種情況會報錯，因為擴充套件運算子所在的括號不是函式呼叫。

### 替代函式的 apply 方法

由於擴充套件運算子可以展開陣列，所以不再需要`apply`方法，將陣列轉為函式的引數了。

```javascript
// ES5 的寫法
function f(x, y, z) {
  // ...
}
var args = [0, 1, 2];
f.apply(null, args);

// ES6的寫法
function f(x, y, z) {
  // ...
}
let args = [0, 1, 2];
f(...args);
```

下面是擴充套件運算子取代`apply`方法的一個實際的例子，應用`Math.max`方法，簡化求出一個數組最大元素的寫法。

```javascript
// ES5 的寫法
Math.max.apply(null, [14, 3, 77])

// ES6 的寫法
Math.max(...[14, 3, 77])

// 等同於
Math.max(14, 3, 77);
```

上面程式碼中，由於 JavaScript 不提供求陣列最大元素的函式，所以只能套用`Math.max`函式，將陣列轉為一個引數序列，然後求最大值。有了擴充套件運算子以後，就可以直接用`Math.max`了。

另一個例子是透過`push`函式，將一個數組新增到另一個數組的尾部。

```javascript
// ES5的 寫法
var arr1 = [0, 1, 2];
var arr2 = [3, 4, 5];
Array.prototype.push.apply(arr1, arr2);

// ES6 的寫法
let arr1 = [0, 1, 2];
let arr2 = [3, 4, 5];
arr1.push(...arr2);
```

上面程式碼的 ES5 寫法中，`push`方法的引數不能是陣列，所以只好透過`apply`方法變通使用`push`方法。有了擴充套件運算子，就可以直接將陣列傳入`push`方法。

下面是另外一個例子。

```javascript
// ES5
new (Date.bind.apply(Date, [null, 2015, 1, 1]))
// ES6
new Date(...[2015, 1, 1]);
```

### 擴充套件運算子的應用

**（1）複製陣列**

陣列是複合的資料型別，直接複製的話，只是複製了指向底層資料結構的指標，而不是克隆一個全新的陣列。

```javascript
const a1 = [1, 2];
const a2 = a1;

a2[0] = 2;
a1 // [2, 2]
```

上面程式碼中，`a2`並不是`a1`的克隆，而是指向同一份資料的另一個指標。修改`a2`，會直接導致`a1`的變化。

ES5 只能用變通方法來複制陣列。

```javascript
const a1 = [1, 2];
const a2 = a1.concat();

a2[0] = 2;
a1 // [1, 2]
```

上面程式碼中，`a1`會返回原陣列的克隆，再修改`a2`就不會對`a1`產生影響。

擴充套件運算子提供了複製陣列的簡便寫法。

```javascript
const a1 = [1, 2];
// 寫法一
const a2 = [...a1];
// 寫法二
const [...a2] = a1;
```

上面的兩種寫法，`a2`都是`a1`的克隆。

**（2）合併陣列**

擴充套件運算子提供了數組合並的新寫法。

```javascript
const arr1 = ['a', 'b'];
const arr2 = ['c'];
const arr3 = ['d', 'e'];

// ES5 的合併陣列
arr1.concat(arr2, arr3);
// [ 'a', 'b', 'c', 'd', 'e' ]

// ES6 的合併陣列
[...arr1, ...arr2, ...arr3]
// [ 'a', 'b', 'c', 'd', 'e' ]
```

不過，這兩種方法都是淺複製，使用的時候需要注意。

```javascript
const a1 = [{ foo: 1 }];
const a2 = [{ bar: 2 }];

const a3 = a1.concat(a2);
const a4 = [...a1, ...a2];

a3[0] === a1[0] // true
a4[0] === a1[0] // true
```

上面程式碼中，`a3`和`a4`是用兩種不同方法合併而成的新陣列，但是它們的成員都是對原陣列成員的引用，這就是淺複製。如果修改了引用指向的值，會同步反映到新陣列。

**（3）與解構賦值結合**

擴充套件運算子可以與解構賦值結合起來，用於生成陣列。

```javascript
// ES5
a = list[0], rest = list.slice(1)
// ES6
[a, ...rest] = list
```

下面是另外一些例子。

```javascript
const [first, ...rest] = [1, 2, 3, 4, 5];
first // 1
rest  // [2, 3, 4, 5]

const [first, ...rest] = [];
first // undefined
rest  // []

const [first, ...rest] = ["foo"];
first  // "foo"
rest   // []
```

如果將擴充套件運算子用於陣列賦值，只能放在引數的最後一位，否則會報錯。

```javascript
const [...butLast, last] = [1, 2, 3, 4, 5];
// 報錯

const [first, ...middle, last] = [1, 2, 3, 4, 5];
// 報錯
```

**（4）字串**

擴充套件運算子還可以將字串轉為真正的陣列。

```javascript
[...'hello']
// [ "h", "e", "l", "l", "o" ]
```

上面的寫法，有一個重要的好處，那就是能夠正確識別四個位元組的 Unicode 字元。

```javascript
'x\uD83D\uDE80y'.length // 4
[...'x\uD83D\uDE80y'].length // 3
```

上面程式碼的第一種寫法，JavaScript 會將四個位元組的 Unicode 字元，識別為 2 個字元，採用擴充套件運算子就沒有這個問題。因此，正確返回字串長度的函式，可以像下面這樣寫。

```javascript
function length(str) {
  return [...str].length;
}

length('x\uD83D\uDE80y') // 3
```

凡是涉及到操作四個位元組的 Unicode 字元的函式，都有這個問題。因此，最好都用擴充套件運算子改寫。

```javascript
let str = 'x\uD83D\uDE80y';

str.split('').reverse().join('')
// 'y\uDE80\uD83Dx'

[...str].reverse().join('')
// 'y\uD83D\uDE80x'
```

上面程式碼中，如果不用擴充套件運算子，字串的`reverse`操作就不正確。

**（5）實現了 Iterator 介面的物件**

任何定義了遍歷器（Iterator）介面的物件（參閱 Iterator 一章），都可以用擴充套件運算子轉為真正的陣列。

```javascript
let nodeList = document.querySelectorAll('div');
let array = [...nodeList];
```

上面程式碼中，`querySelectorAll`方法返回的是一個`NodeList`物件。它不是陣列，而是一個類似陣列的物件。這時，擴充套件運算子可以將其轉為真正的陣列，原因就在於`NodeList`物件實現了 Iterator 。

```javascript
Number.prototype[Symbol.iterator] = function*() {
  let i = 0;
  let num = this.valueOf();
  while (i < num) {
    yield i++;
  }
}

console.log([...5]) // [0, 1, 2, 3, 4]
```

上面程式碼中，先定義了`Number`物件的遍歷器介面，擴充套件運算子將`5`自動轉成`Number`例項以後，就會呼叫這個介面，就會返回自定義的結果。

對於那些沒有部署 Iterator 介面的類似陣列的物件，擴充套件運算子就無法將其轉為真正的陣列。

```javascript
let arrayLike = {
  '0': 'a',
  '1': 'b',
  '2': 'c',
  length: 3
};

// TypeError: Cannot spread non-iterable object.
let arr = [...arrayLike];
```

上面程式碼中，`arrayLike`是一個類似陣列的物件，但是沒有部署 Iterator 介面，擴充套件運算子就會報錯。這時，可以改為使用`Array.from`方法將`arrayLike`轉為真正的陣列。

**（6）Map 和 Set 結構，Generator 函式**

擴充套件運算子內部呼叫的是資料結構的 Iterator 介面，因此只要具有 Iterator 介面的物件，都可以使用擴充套件運算子，比如 Map 結構。

```javascript
let map = new Map([
  [1, 'one'],
  [2, 'two'],
  [3, 'three'],
]);

let arr = [...map.keys()]; // [1, 2, 3]
```

Generator 函式執行後，返回一個遍歷器物件，因此也可以使用擴充套件運算子。

```javascript
const go = function*(){
  yield 1;
  yield 2;
  yield 3;
};

[...go()] // [1, 2, 3]
```

上面程式碼中，變數`go`是一個 Generator 函式，執行後返回的是一個遍歷器物件，對這個遍歷器物件執行擴充套件運算子，就會將內部遍歷得到的值，轉為一個數組。

如果對沒有 Iterator 介面的物件，使用擴充套件運算子，將會報錯。

```javascript
const obj = {a: 1, b: 2};
let arr = [...obj]; // TypeError: Cannot spread non-iterable object
```

## Array.from()

`Array.from`方法用於將兩類物件轉為真正的陣列：類似陣列的物件（array-like object）和可遍歷（iterable）的物件（包括 ES6 新增的資料結構 Set 和 Map）。

下面是一個類似陣列的物件，`Array.from`將它轉為真正的陣列。

```javascript
let arrayLike = {
    '0': 'a',
    '1': 'b',
    '2': 'c',
    length: 3
};

// ES5的寫法
var arr1 = [].slice.call(arrayLike); // ['a', 'b', 'c']

// ES6的寫法
let arr2 = Array.from(arrayLike); // ['a', 'b', 'c']
```

實際應用中，常見的類似陣列的物件是 DOM 操作返回的 NodeList 集合，以及函式內部的`arguments`物件。`Array.from`都可以將它們轉為真正的陣列。

```javascript
// NodeList物件
let ps = document.querySelectorAll('p');
Array.from(ps).filter(p => {
  return p.textContent.length > 100;
});

// arguments物件
function foo() {
  var args = Array.from(arguments);
  // ...
}
```

上面程式碼中，`querySelectorAll`方法返回的是一個類似陣列的物件，可以將這個物件轉為真正的陣列，再使用`filter`方法。

只要是部署了 Iterator 介面的資料結構，`Array.from`都能將其轉為陣列。

```javascript
Array.from('hello')
// ['h', 'e', 'l', 'l', 'o']

let namesSet = new Set(['a', 'b'])
Array.from(namesSet) // ['a', 'b']
```

上面程式碼中，字串和 Set 結構都具有 Iterator 介面，因此可以被`Array.from`轉為真正的陣列。

如果引數是一個真正的陣列，`Array.from`會返回一個一模一樣的新陣列。

```javascript
Array.from([1, 2, 3])
// [1, 2, 3]
```

值得提醒的是，擴充套件運算子（`...`）也可以將某些資料結構轉為陣列。

```javascript
// arguments物件
function foo() {
  const args = [...arguments];
}

// NodeList物件
[...document.querySelectorAll('div')]
```

擴充套件運算子背後呼叫的是遍歷器介面（`Symbol.iterator`），如果一個物件沒有部署這個介面，就無法轉換。`Array.from`方法還支援類似陣列的物件。所謂類似陣列的物件，本質特徵只有一點，即必須有`length`屬性。因此，任何有`length`屬性的物件，都可以透過`Array.from`方法轉為陣列，而此時擴充套件運算子就無法轉換。

```javascript
Array.from({ length: 3 });
// [ undefined, undefined, undefined ]
```

上面程式碼中，`Array.from`返回了一個具有三個成員的陣列，每個位置的值都是`undefined`。擴充套件運算子轉換不了這個物件。

對於還沒有部署該方法的瀏覽器，可以用`Array.prototype.slice`方法替代。

```javascript
const toArray = (() =>
  Array.from ? Array.from : obj => [].slice.call(obj)
)();
```

`Array.from`還可以接受第二個引數，作用類似於陣列的`map`方法，用來對每個元素進行處理，將處理後的值放入返回的陣列。

```javascript
Array.from(arrayLike, x => x * x);
// 等同於
Array.from(arrayLike).map(x => x * x);

Array.from([1, 2, 3], (x) => x * x)
// [1, 4, 9]
```

下面的例子是取出一組 DOM 節點的文字內容。

```javascript
let spans = document.querySelectorAll('span.name');

// map()
let names1 = Array.prototype.map.call(spans, s => s.textContent);

// Array.from()
let names2 = Array.from(spans, s => s.textContent)
```

下面的例子將陣列中布林值為`false`的成員轉為`0`。

```javascript
Array.from([1, , 2, , 3], (n) => n || 0)
// [1, 0, 2, 0, 3]
```

另一個例子是返回各種資料的型別。

```javascript
function typesOf () {
  return Array.from(arguments, value => typeof value)
}
typesOf(null, [], NaN)
// ['object', 'object', 'number']
```

如果`map`函式裡面用到了`this`關鍵字，還可以傳入`Array.from`的第三個引數，用來繫結`this`。

`Array.from()`可以將各種值轉為真正的陣列，並且還提供`map`功能。這實際上意味著，只要有一個原始的資料結構，你就可以先對它的值進行處理，然後轉成規範的陣列結構，進而就可以使用數量眾多的陣列方法。

```javascript
Array.from({ length: 2 }, () => 'jack')
// ['jack', 'jack']
```

上面程式碼中，`Array.from`的第一個引數指定了第二個引數執行的次數。這種特性可以讓該方法的用法變得非常靈活。

`Array.from()`的另一個應用是，將字串轉為陣列，然後返回字串的長度。因為它能正確處理各種 Unicode 字元，可以避免 JavaScript 將大於`\uFFFF`的 Unicode 字元，算作兩個字元的 bug。

```javascript
function countSymbols(string) {
  return Array.from(string).length;
}
```

## Array.of()

`Array.of`方法用於將一組值，轉換為陣列。

```javascript
Array.of(3, 11, 8) // [3,11,8]
Array.of(3) // [3]
Array.of(3).length // 1
```

這個方法的主要目的，是彌補陣列建構函式`Array()`的不足。因為引數個數的不同，會導致`Array()`的行為有差異。

```javascript
Array() // []
Array(3) // [, , ,]
Array(3, 11, 8) // [3, 11, 8]
```

上面程式碼中，`Array`方法沒有引數、一個引數、三個引數時，返回結果都不一樣。只有當引數個數不少於 2 個時，`Array()`才會返回由引數組成的新陣列。引數個數只有一個時，實際上是指定陣列的長度。

`Array.of`基本上可以用來替代`Array()`或`new Array()`，並且不存在由於引數不同而導致的過載。它的行為非常統一。

```javascript
Array.of() // []
Array.of(undefined) // [undefined]
Array.of(1) // [1]
Array.of(1, 2) // [1, 2]
```

`Array.of`總是返回引數值組成的陣列。如果沒有引數，就返回一個空陣列。

`Array.of`方法可以用下面的程式碼模擬實現。

```javascript
function ArrayOf(){
  return [].slice.call(arguments);
}
```

## 陣列例項的 copyWithin()

陣列例項的`copyWithin()`方法，在當前陣列內部，將指定位置的成員複製到其他位置（會覆蓋原有成員），然後返回當前陣列。也就是說，使用這個方法，會修改當前陣列。

```javascript
Array.prototype.copyWithin(target, start = 0, end = this.length)
```

它接受三個引數。

- target（必需）：從該位置開始替換資料。如果為負值，表示倒數。
- start（可選）：從該位置開始讀取資料，預設為 0。如果為負值，表示從末尾開始計算。
- end（可選）：到該位置前停止讀取資料，預設等於陣列長度。如果為負值，表示從末尾開始計算。

這三個引數都應該是數值，如果不是，會自動轉為數值。

```javascript
[1, 2, 3, 4, 5].copyWithin(0, 3)
// [4, 5, 3, 4, 5]
```

上面程式碼表示將從 3 號位直到陣列結束的成員（4 和 5），複製到從 0 號位開始的位置，結果覆蓋了原來的 1 和 2。

下面是更多例子。

```javascript
// 將3號位複製到0號位
[1, 2, 3, 4, 5].copyWithin(0, 3, 4)
// [4, 2, 3, 4, 5]

// -2相當於3號位，-1相當於4號位
[1, 2, 3, 4, 5].copyWithin(0, -2, -1)
// [4, 2, 3, 4, 5]

// 將3號位複製到0號位
[].copyWithin.call({length: 5, 3: 1}, 0, 3)
// {0: 1, 3: 1, length: 5}

// 將2號位到陣列結束，複製到0號位
let i32a = new Int32Array([1, 2, 3, 4, 5]);
i32a.copyWithin(0, 2);
// Int32Array [3, 4, 5, 4, 5]

// 對於沒有部署 TypedArray 的 copyWithin 方法的平臺
// 需要採用下面的寫法
[].copyWithin.call(new Int32Array([1, 2, 3, 4, 5]), 0, 3, 4);
// Int32Array [4, 2, 3, 4, 5]
```

## 陣列例項的 find() 和 findIndex()

陣列例項的`find`方法，用於找出第一個符合條件的陣列成員。它的引數是一個回撥函式，所有陣列成員依次執行該回調函式，直到找出第一個返回值為`true`的成員，然後返回該成員。如果沒有符合條件的成員，則返回`undefined`。

```javascript
[1, 4, -5, 10].find((n) => n < 0)
// -5
```

上面程式碼找出陣列中第一個小於 0 的成員。

```javascript
[1, 5, 10, 15].find(function(value, index, arr) {
  return value > 9;
}) // 10
```

上面程式碼中，`find`方法的回撥函式可以接受三個引數，依次為當前的值、當前的位置和原陣列。

陣列例項的`findIndex`方法的用法與`find`方法非常類似，返回第一個符合條件的陣列成員的位置，如果所有成員都不符合條件，則返回`-1`。

```javascript
[1, 5, 10, 15].findIndex(function(value, index, arr) {
  return value > 9;
}) // 2
```

這兩個方法都可以接受第二個引數，用來繫結回撥函式的`this`物件。

```javascript
function f(v){
  return v > this.age;
}
let person = {name: 'John', age: 20};
[10, 12, 26, 15].find(f, person);    // 26
```

上面的程式碼中，`find`函式接收了第二個引數`person`物件，回撥函式中的`this`物件指向`person`物件。

另外，這兩個方法都可以發現`NaN`，彌補了陣列的`indexOf`方法的不足。

```javascript
[NaN].indexOf(NaN)
// -1

[NaN].findIndex(y => Object.is(NaN, y))
// 0
```

上面程式碼中，`indexOf`方法無法識別陣列的`NaN`成員，但是`findIndex`方法可以藉助`Object.is`方法做到。

## 陣列例項的 fill()

`fill`方法使用給定值，填充一個數組。

```javascript
['a', 'b', 'c'].fill(7)
// [7, 7, 7]

new Array(3).fill(7)
// [7, 7, 7]
```

上面程式碼表明，`fill`方法用於空陣列的初始化非常方便。陣列中已有的元素，會被全部抹去。

`fill`方法還可以接受第二個和第三個引數，用於指定填充的起始位置和結束位置。

```javascript
['a', 'b', 'c'].fill(7, 1, 2)
// ['a', 7, 'c']
```

上面程式碼表示，`fill`方法從 1 號位開始，向原陣列填充 7，到 2 號位之前結束。

注意，如果填充的型別為物件，那麼被賦值的是同一個記憶體地址的物件，而不是深複製物件。

```javascript
let arr = new Array(3).fill({name: "Mike"});
arr[0].name = "Ben";
arr
// [{name: "Ben"}, {name: "Ben"}, {name: "Ben"}]

let arr = new Array(3).fill([]);
arr[0].push(5);
arr
// [[5], [5], [5]]
```

## 陣列例項的 entries()，keys() 和 values()

ES6 提供三個新的方法——`entries()`，`keys()`和`values()`——用於遍歷陣列。它們都返回一個遍歷器物件（詳見《Iterator》一章），可以用`for...of`迴圈進行遍歷，唯一的區別是`keys()`是對鍵名的遍歷、`values()`是對鍵值的遍歷，`entries()`是對鍵值對的遍歷。

```javascript
for (let index of ['a', 'b'].keys()) {
  console.log(index);
}
// 0
// 1

for (let elem of ['a', 'b'].values()) {
  console.log(elem);
}
// 'a'
// 'b'

for (let [index, elem] of ['a', 'b'].entries()) {
  console.log(index, elem);
}
// 0 "a"
// 1 "b"
```

如果不使用`for...of`迴圈，可以手動呼叫遍歷器物件的`next`方法，進行遍歷。

```javascript
let letter = ['a', 'b', 'c'];
let entries = letter.entries();
console.log(entries.next().value); // [0, 'a']
console.log(entries.next().value); // [1, 'b']
console.log(entries.next().value); // [2, 'c']
```

## 陣列例項的 includes()

`Array.prototype.includes`方法返回一個布林值，表示某個陣列是否包含給定的值，與字串的`includes`方法類似。ES2016 引入了該方法。

```javascript
[1, 2, 3].includes(2)     // true
[1, 2, 3].includes(4)     // false
[1, 2, NaN].includes(NaN) // true
```

該方法的第二個引數表示搜尋的起始位置，預設為`0`。如果第二個引數為負數，則表示倒數的位置，如果這時它大於陣列長度（比如第二個引數為`-4`，但陣列長度為`3`），則會重置為從`0`開始。

```javascript
[1, 2, 3].includes(3, 3);  // false
[1, 2, 3].includes(3, -1); // true
```

沒有該方法之前，我們通常使用陣列的`indexOf`方法，檢查是否包含某個值。

```javascript
if (arr.indexOf(el) !== -1) {
  // ...
}
```

`indexOf`方法有兩個缺點，一是不夠語義化，它的含義是找到引數值的第一個出現位置，所以要去比較是否不等於`-1`，表達起來不夠直觀。二是，它內部使用嚴格相等運算子（`===`）進行判斷，這會導致對`NaN`的誤判。

```javascript
[NaN].indexOf(NaN)
// -1
```

`includes`使用的是不一樣的判斷演算法，就沒有這個問題。

```javascript
[NaN].includes(NaN)
// true
```

下面程式碼用來檢查當前環境是否支援該方法，如果不支援，部署一個簡易的替代版本。

```javascript
const contains = (() =>
  Array.prototype.includes
    ? (arr, value) => arr.includes(value)
    : (arr, value) => arr.some(el => el === value)
)();
contains(['foo', 'bar'], 'baz'); // => false
```

另外，Map 和 Set 資料結構有一個`has`方法，需要注意與`includes`區分。

- Map 結構的`has`方法，是用來查詢鍵名的，比如`Map.prototype.has(key)`、`WeakMap.prototype.has(key)`、`Reflect.has(target, propertyKey)`。
- Set 結構的`has`方法，是用來查詢值的，比如`Set.prototype.has(value)`、`WeakSet.prototype.has(value)`。

## 陣列例項的 flat()，flatMap()

陣列的成員有時還是陣列，`Array.prototype.flat()`用於將巢狀的陣列“拉平”，變成一維的陣列。該方法返回一個新陣列，對原資料沒有影響。

```javascript
[1, 2, [3, 4]].flat()
// [1, 2, 3, 4]
```

上面程式碼中，原陣列的成員裡面有一個數組，`flat()`方法將子陣列的成員取出來，新增在原來的位置。

`flat()`預設只會“拉平”一層，如果想要“拉平”多層的巢狀陣列，可以將`flat()`方法的引數寫成一個整數，表示想要拉平的層數，預設為1。

```javascript
[1, 2, [3, [4, 5]]].flat()
// [1, 2, 3, [4, 5]]

[1, 2, [3, [4, 5]]].flat(2)
// [1, 2, 3, 4, 5]
```

上面程式碼中，`flat()`的引數為2，表示要“拉平”兩層的巢狀陣列。

如果不管有多少層巢狀，都要轉成一維陣列，可以用`Infinity`關鍵字作為引數。

```javascript
[1, [2, [3]]].flat(Infinity)
// [1, 2, 3]
```

如果原陣列有空位，`flat()`方法會跳過空位。

```javascript
[1, 2, , 4, 5].flat()
// [1, 2, 4, 5]
```

`flatMap()`方法對原陣列的每個成員執行一個函式（相當於執行`Array.prototype.map()`），然後對返回值組成的陣列執行`flat()`方法。該方法返回一個新陣列，不改變原陣列。

```javascript
// 相當於 [[2, 4], [3, 6], [4, 8]].flat()
[2, 3, 4].flatMap((x) => [x, x * 2])
// [2, 4, 3, 6, 4, 8]
```

`flatMap()`只能展開一層陣列。

```javascript
// 相當於 [[[2]], [[4]], [[6]], [[8]]].flat()
[1, 2, 3, 4].flatMap(x => [[x * 2]])
// [[2], [4], [6], [8]]
```

上面程式碼中，遍歷函式返回的是一個雙層的陣列，但是預設只能展開一層，因此`flatMap()`返回的還是一個巢狀陣列。

`flatMap()`方法的引數是一個遍歷函式，該函式可以接受三個引數，分別是當前陣列成員、當前陣列成員的位置（從零開始）、原陣列。

```javascript
arr.flatMap(function callback(currentValue[, index[, array]]) {
  // ...
}[, thisArg])
```

`flatMap()`方法還可以有第二個引數，用來繫結遍歷函式裡面的`this`。

## 陣列的空位

陣列的空位指，陣列的某一個位置沒有任何值。比如，`Array`建構函式返回的陣列都是空位。

```javascript
Array(3) // [, , ,]
```

上面程式碼中，`Array(3)`返回一個具有 3 個空位的陣列。

注意，空位不是`undefined`，一個位置的值等於`undefined`，依然是有值的。空位是沒有任何值，`in`運算子可以說明這一點。

```javascript
0 in [undefined, undefined, undefined] // true
0 in [, , ,] // false
```

上面程式碼說明，第一個陣列的 0 號位置是有值的，第二個陣列的 0 號位置沒有值。

ES5 對空位的處理，已經很不一致了，大多數情況下會忽略空位。

- `forEach()`, `filter()`, `reduce()`, `every()` 和`some()`都會跳過空位。
- `map()`會跳過空位，但會保留這個值
- `join()`和`toString()`會將空位視為`undefined`，而`undefined`和`null`會被處理成空字串。

```javascript
// forEach方法
[,'a'].forEach((x,i) => console.log(i)); // 1

// filter方法
['a',,'b'].filter(x => true) // ['a','b']

// every方法
[,'a'].every(x => x==='a') // true

// reduce方法
[1,,2].reduce((x,y) => x+y) // 3

// some方法
[,'a'].some(x => x !== 'a') // false

// map方法
[,'a'].map(x => 1) // [,1]

// join方法
[,'a',undefined,null].join('#') // "#a##"

// toString方法
[,'a',undefined,null].toString() // ",a,,"
```

ES6 則是明確將空位轉為`undefined`。

`Array.from`方法會將陣列的空位，轉為`undefined`，也就是說，這個方法不會忽略空位。

```javascript
Array.from(['a',,'b'])
// [ "a", undefined, "b" ]
```

擴充套件運算子（`...`）也會將空位轉為`undefined`。

```javascript
[...['a',,'b']]
// [ "a", undefined, "b" ]
```

`copyWithin()`會連空位一起複製。

```javascript
[,'a','b',,].copyWithin(2,0) // [,"a",,"a"]
```

`fill()`會將空位視為正常的陣列位置。

```javascript
new Array(3).fill('a') // ["a","a","a"]
```

`for...of`迴圈也會遍歷空位。

```javascript
let arr = [, ,];
for (let i of arr) {
  console.log(1);
}
// 1
// 1
```

上面程式碼中，陣列`arr`有兩個空位，`for...of`並沒有忽略它們。如果改成`map`方法遍歷，空位是會跳過的。

`entries()`、`keys()`、`values()`、`find()`和`findIndex()`會將空位處理成`undefined`。

```javascript
// entries()
[...[,'a'].entries()] // [[0,undefined], [1,"a"]]

// keys()
[...[,'a'].keys()] // [0,1]

// values()
[...[,'a'].values()] // [undefined,"a"]

// find()
[,'a'].find(x => true) // undefined

// findIndex()
[,'a'].findIndex(x => true) // 0
```

由於空位的處理規則非常不統一，所以建議避免出現空位。

## Array.prototype.sort() 的排序穩定性

排序穩定性（stable sorting）是排序演算法的重要屬性，指的是排序關鍵字相同的專案，排序前後的順序不變。

```javascript
const arr = [
  'peach',
  'straw',
  'apple',
  'spork'
];

const stableSorting = (s1, s2) => {
  if (s1[0] < s2[0]) return -1;
  return 1;
};

arr.sort(stableSorting)
// ["apple", "peach", "straw", "spork"]
```

上面程式碼對陣列`arr`按照首字母進行排序。排序結果中，`straw`在`spork`的前面，跟原始順序一致，所以排序演算法`stableSorting`是穩定排序。

```javascript
const unstableSorting = (s1, s2) => {
  if (s1[0] <= s2[0]) return -1;
  return 1;
};

arr.sort(unstableSorting)
// ["apple", "peach", "spork", "straw"]
```

上面程式碼中，排序結果是`spork`在`straw`前面，跟原始順序相反，所以排序演算法`unstableSorting`是不穩定的。

常見的排序演算法之中，插入排序、合併排序、氣泡排序等都是穩定的，堆排序、快速排序等是不穩定的。不穩定排序的主要缺點是，多重排序時可能會產生問題。假設有一個姓和名的列表，要求按照“姓氏為主要關鍵字，名字為次要關鍵字”進行排序。開發者可能會先按名字排序，再按姓氏進行排序。如果排序演算法是穩定的，這樣就可以達到“先姓氏，後名字”的排序效果。如果是不穩定的，就不行。

早先的 ECMAScript 沒有規定，`Array.prototype.sort()`的預設排序演算法是否穩定，留給瀏覽器自己決定，這導致某些實現是不穩定的。[ES2019](https://github.com/tc39/ecma262/pull/1340) 明確規定，`Array.prototype.sort()`的預設排序演算法必須穩定。這個規定已經做到了，現在 JavaScript 各個主要實現的預設排序演算法都是穩定的。

