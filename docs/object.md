# 物件的擴充套件

物件（object）是 JavaScript 最重要的資料結構。ES6 對它進行了重大升級，本章介紹資料結構本身的改變，下一章介紹`Object`物件的新增方法。

## 屬性的簡潔表示法

ES6 允許在大括號裡面，直接寫入變數和函式，作為物件的屬性和方法。這樣的書寫更加簡潔。

```javascript
const foo = 'bar';
const baz = {foo};
baz // {foo: "bar"}

// 等同於
const baz = {foo: foo};
```

上面程式碼中，變數`foo`直接寫在大括號裡面。這時，屬性名就是變數名, 屬性值就是變數值。下面是另一個例子。

```javascript
function f(x, y) {
  return {x, y};
}

// 等同於

function f(x, y) {
  return {x: x, y: y};
}

f(1, 2) // Object {x: 1, y: 2}
```

除了屬性簡寫，方法也可以簡寫。

```javascript
const o = {
  method() {
    return "Hello!";
  }
};

// 等同於

const o = {
  method: function() {
    return "Hello!";
  }
};
```

下面是一個實際的例子。

```javascript
let birth = '2000/01/01';

const Person = {

  name: '張三',

  //等同於birth: birth
  birth,

  // 等同於hello: function ()...
  hello() { console.log('我的名字是', this.name); }

};
```

這種寫法用於函式的返回值，將會非常方便。

```javascript
function getPoint() {
  const x = 1;
  const y = 10;
  return {x, y};
}

getPoint()
// {x:1, y:10}
```

CommonJS 模組輸出一組變數，就非常合適使用簡潔寫法。

```javascript
let ms = {};

function getItem (key) {
  return key in ms ? ms[key] : null;
}

function setItem (key, value) {
  ms[key] = value;
}

function clear () {
  ms = {};
}

module.exports = { getItem, setItem, clear };
// 等同於
module.exports = {
  getItem: getItem,
  setItem: setItem,
  clear: clear
};
```

屬性的賦值器（setter）和取值器（getter），事實上也是採用這種寫法。

```javascript
const cart = {
  _wheels: 4,

  get wheels () {
    return this._wheels;
  },

  set wheels (value) {
    if (value < this._wheels) {
      throw new Error('數值太小了！');
    }
    this._wheels = value;
  }
}
```

簡潔寫法在列印物件時也很有用。

```javascript
let user = {
  name: 'test'
};

let foo = {
  bar: 'baz'
};

console.log(user, foo)
// {name: "test"} {bar: "baz"}
console.log({user, foo})
// {user: {name: "test"}, foo: {bar: "baz"}}
```

上面程式碼中，`console.log`直接輸出`user`和`foo`兩個物件時，就是兩組鍵值對，可能會混淆。把它們放在大括號裡面輸出，就變成了物件的簡潔表示法，每組鍵值對前面會列印物件名，這樣就比較清晰了。

注意，簡寫的物件方法不能用作建構函式，會報錯。

```javascript
const obj = {
  f() {
    this.foo = 'bar';
  }
};

new obj.f() // 報錯
```

上面程式碼中，`f`是一個簡寫的物件方法，所以`obj.f`不能當作建構函式使用。

## 屬性名錶達式

JavaScript 定義物件的屬性，有兩種方法。

```javascript
// 方法一
obj.foo = true;

// 方法二
obj['a' + 'bc'] = 123;
```

上面程式碼的方法一是直接用識別符號作為屬性名，方法二是用表示式作為屬性名，這時要將表示式放在方括號之內。

但是，如果使用字面量方式定義物件（使用大括號），在 ES5 中只能使用方法一（識別符號）定義屬性。

```javascript
var obj = {
  foo: true,
  abc: 123
};
```

ES6 允許字面量定義物件時，用方法二（表示式）作為物件的屬性名，即把表示式放在方括號內。

```javascript
let propKey = 'foo';

let obj = {
  [propKey]: true,
  ['a' + 'bc']: 123
};
```

下面是另一個例子。

```javascript
let lastWord = 'last word';

const a = {
  'first word': 'hello',
  [lastWord]: 'world'
};

a['first word'] // "hello"
a[lastWord] // "world"
a['last word'] // "world"
```

表示式還可以用於定義方法名。

```javascript
let obj = {
  ['h' + 'ello']() {
    return 'hi';
  }
};

obj.hello() // hi
```

注意，屬性名錶達式與簡潔表示法，不能同時使用，會報錯。

```javascript
// 報錯
const foo = 'bar';
const bar = 'abc';
const baz = { [foo] };

// 正確
const foo = 'bar';
const baz = { [foo]: 'abc'};
```

注意，屬性名錶達式如果是一個物件，預設情況下會自動將物件轉為字串`[object Object]`，這一點要特別小心。

```javascript
const keyA = {a: 1};
const keyB = {b: 2};

const myObject = {
  [keyA]: 'valueA',
  [keyB]: 'valueB'
};

myObject // Object {[object Object]: "valueB"}
```

上面程式碼中，`[keyA]`和`[keyB]`得到的都是`[object Object]`，所以`[keyB]`會把`[keyA]`覆蓋掉，而`myObject`最後只有一個`[object Object]`屬性。

## 方法的 name 屬性

函式的`name`屬性，返回函式名。物件方法也是函式，因此也有`name`屬性。

```javascript
const person = {
  sayName() {
    console.log('hello!');
  },
};

person.sayName.name   // "sayName"
```

上面程式碼中，方法的`name`屬性返回函式名（即方法名）。

如果物件的方法使用了取值函式（`getter`）和存值函式（`setter`），則`name`屬性不是在該方法上面，而是該方法的屬性的描述物件的`get`和`set`屬性上面，返回值是方法名前加上`get`和`set`。

```javascript
const obj = {
  get foo() {},
  set foo(x) {}
};

obj.foo.name
// TypeError: Cannot read property 'name' of undefined

const descriptor = Object.getOwnPropertyDescriptor(obj, 'foo');

descriptor.get.name // "get foo"
descriptor.set.name // "set foo"
```

有兩種特殊情況：`bind`方法創造的函式，`name`屬性返回`bound`加上原函式的名字；`Function`建構函式創造的函式，`name`屬性返回`anonymous`。

```javascript
(new Function()).name // "anonymous"

var doSomething = function() {
  // ...
};
doSomething.bind().name // "bound doSomething"
```

如果物件的方法是一個 Symbol 值，那麼`name`屬性返回的是這個 Symbol 值的描述。

```javascript
const key1 = Symbol('description');
const key2 = Symbol();
let obj = {
  [key1]() {},
  [key2]() {},
};
obj[key1].name // "[description]"
obj[key2].name // ""
```

上面程式碼中，`key1`對應的 Symbol 值有描述，`key2`沒有。

## 屬性的可列舉性和遍歷

### 可列舉性

物件的每個屬性都有一個描述物件（Descriptor），用來控制該屬性的行為。`Object.getOwnPropertyDescriptor`方法可以獲取該屬性的描述物件。

```javascript
let obj = { foo: 123 };
Object.getOwnPropertyDescriptor(obj, 'foo')
//  {
//    value: 123,
//    writable: true,
//    enumerable: true,
//    configurable: true
//  }
```

描述物件的`enumerable`屬性，稱為“可列舉性”，如果該屬性為`false`，就表示某些操作會忽略當前屬性。

目前，有四個操作會忽略`enumerable`為`false`的屬性。

- `for...in`迴圈：只遍歷物件自身的和繼承的可列舉的屬性。
- `Object.keys()`：返回物件自身的所有可列舉的屬性的鍵名。
- `JSON.stringify()`：只序列化物件自身的可列舉的屬性。
- `Object.assign()`： 忽略`enumerable`為`false`的屬性，只複製物件自身的可列舉的屬性。

這四個操作之中，前三個是 ES5 就有的，最後一個`Object.assign()`是 ES6 新增的。其中，只有`for...in`會返回繼承的屬性，其他三個方法都會忽略繼承的屬性，只處理物件自身的屬性。實際上，引入“可列舉”（`enumerable`）這個概念的最初目的，就是讓某些屬性可以規避掉`for...in`操作，不然所有內部屬性和方法都會被遍歷到。比如，物件原型的`toString`方法，以及陣列的`length`屬性，就透過“可列舉性”，從而避免被`for...in`遍歷到。

```javascript
Object.getOwnPropertyDescriptor(Object.prototype, 'toString').enumerable
// false

Object.getOwnPropertyDescriptor([], 'length').enumerable
// false
```

上面程式碼中，`toString`和`length`屬性的`enumerable`都是`false`，因此`for...in`不會遍歷到這兩個繼承自原型的屬性。

另外，ES6 規定，所有 Class 的原型的方法都是不可列舉的。

```javascript
Object.getOwnPropertyDescriptor(class {foo() {}}.prototype, 'foo').enumerable
// false
```

總的來說，操作中引入繼承的屬性會讓問題複雜化，大多數時候，我們只關心物件自身的屬性。所以，儘量不要用`for...in`迴圈，而用`Object.keys()`代替。

### 屬性的遍歷

ES6 一共有 5 種方法可以遍歷物件的屬性。

**（1）for...in**

`for...in`迴圈遍歷物件自身的和繼承的可列舉屬性（不含 Symbol 屬性）。

**（2）Object.keys(obj)**

`Object.keys`返回一個數組，包括物件自身的（不含繼承的）所有可列舉屬性（不含 Symbol 屬性）的鍵名。

**（3）Object.getOwnPropertyNames(obj)**

`Object.getOwnPropertyNames`返回一個數組，包含物件自身的所有屬性（不含 Symbol 屬性，但是包括不可列舉屬性）的鍵名。

**（4）Object.getOwnPropertySymbols(obj)**

`Object.getOwnPropertySymbols`返回一個數組，包含物件自身的所有 Symbol 屬性的鍵名。

**（5）Reflect.ownKeys(obj)**

`Reflect.ownKeys`返回一個數組，包含物件自身的（不含繼承的）所有鍵名，不管鍵名是 Symbol 或字串，也不管是否可列舉。

以上的 5 種方法遍歷物件的鍵名，都遵守同樣的屬性遍歷的次序規則。

- 首先遍歷所有數值鍵，按照數值升序排列。
- 其次遍歷所有字串鍵，按照加入時間升序排列。
- 最後遍歷所有 Symbol 鍵，按照加入時間升序排列。

```javascript
Reflect.ownKeys({ [Symbol()]:0, b:0, 10:0, 2:0, a:0 })
// ['2', '10', 'b', 'a', Symbol()]
```

上面程式碼中，`Reflect.ownKeys`方法返回一個數組，包含了引數物件的所有屬性。這個陣列的屬性次序是這樣的，首先是數值屬性`2`和`10`，其次是字串屬性`b`和`a`，最後是 Symbol 屬性。

## super 關鍵字

我們知道，`this`關鍵字總是指向函式所在的當前物件，ES6 又新增了另一個類似的關鍵字`super`，指向當前物件的原型物件。

```javascript
const proto = {
  foo: 'hello'
};

const obj = {
  foo: 'world',
  find() {
    return super.foo;
  }
};

Object.setPrototypeOf(obj, proto);
obj.find() // "hello"
```

上面程式碼中，物件`obj.find()`方法之中，透過`super.foo`引用了原型物件`proto`的`foo`屬性。

注意，`super`關鍵字表示原型物件時，只能用在物件的方法之中，用在其他地方都會報錯。

```javascript
// 報錯
const obj = {
  foo: super.foo
}

// 報錯
const obj = {
  foo: () => super.foo
}

// 報錯
const obj = {
  foo: function () {
    return super.foo
  }
}
```

上面三種`super`的用法都會報錯，因為對於 JavaScript 引擎來說，這裡的`super`都沒有用在物件的方法之中。第一種寫法是`super`用在屬性裡面，第二種和第三種寫法是`super`用在一個函式裡面，然後賦值給`foo`屬性。目前，只有物件方法的簡寫法可以讓 JavaScript 引擎確認，定義的是物件的方法。

JavaScript 引擎內部，`super.foo`等同於`Object.getPrototypeOf(this).foo`（屬性）或`Object.getPrototypeOf(this).foo.call(this)`（方法）。

```javascript
const proto = {
  x: 'hello',
  foo() {
    console.log(this.x);
  },
};

const obj = {
  x: 'world',
  foo() {
    super.foo();
  }
}

Object.setPrototypeOf(obj, proto);

obj.foo() // "world"
```

上面程式碼中，`super.foo`指向原型物件`proto`的`foo`方法，但是繫結的`this`卻還是當前物件`obj`，因此輸出的就是`world`。

## 物件的擴充套件運算子

《陣列的擴充套件》一章中，已經介紹過擴充套件運算子（`...`）。ES2018 將這個運算子[引入](https://github.com/sebmarkbage/ecmascript-rest-spread)了物件。

### 解構賦值

物件的解構賦值用於從一個物件取值，相當於將目標物件自身的所有可遍歷的（enumerable）、但尚未被讀取的屬性，分配到指定的物件上面。所有的鍵和它們的值，都會複製到新物件上面。

```javascript
let { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };
x // 1
y // 2
z // { a: 3, b: 4 }
```

上面程式碼中，變數`z`是解構賦值所在的物件。它獲取等號右邊的所有尚未讀取的鍵（`a`和`b`），將它們連同值一起複製過來。

由於解構賦值要求等號右邊是一個物件，所以如果等號右邊是`undefined`或`null`，就會報錯，因為它們無法轉為物件。

```javascript
let { ...z } = null; // 執行時錯誤
let { ...z } = undefined; // 執行時錯誤
```

解構賦值必須是最後一個引數，否則會報錯。

```javascript
let { ...x, y, z } = someObject; // 句法錯誤
let { x, ...y, ...z } = someObject; // 句法錯誤
```

上面程式碼中，解構賦值不是最後一個引數，所以會報錯。

注意，解構賦值的複製是淺複製，即如果一個鍵的值是複合型別的值（陣列、物件、函式）、那麼解構賦值複製的是這個值的引用，而不是這個值的副本。

```javascript
let obj = { a: { b: 1 } };
let { ...x } = obj;
obj.a.b = 2;
x.a.b // 2
```

上面程式碼中，`x`是解構賦值所在的物件，複製了物件`obj`的`a`屬性。`a`屬性引用了一個物件，修改這個物件的值，會影響到解構賦值對它的引用。

另外，擴充套件運算子的解構賦值，不能複製繼承自原型物件的屬性。

```javascript
let o1 = { a: 1 };
let o2 = { b: 2 };
o2.__proto__ = o1;
let { ...o3 } = o2;
o3 // { b: 2 }
o3.a // undefined
```

上面程式碼中，物件`o3`複製了`o2`，但是隻複製了`o2`自身的屬性，沒有複製它的原型物件`o1`的屬性。

下面是另一個例子。

```javascript
const o = Object.create({ x: 1, y: 2 });
o.z = 3;

let { x, ...newObj } = o;
let { y, z } = newObj;
x // 1
y // undefined
z // 3
```

上面程式碼中，變數`x`是單純的解構賦值，所以可以讀取物件`o`繼承的屬性；變數`y`和`z`是擴充套件運算子的解構賦值，只能讀取物件`o`自身的屬性，所以變數`z`可以賦值成功，變數`y`取不到值。ES6 規定，變數宣告語句之中，如果使用解構賦值，擴充套件運算子後面必須是一個變數名，而不能是一個解構賦值表示式，所以上面程式碼引入了中間變數`newObj`，如果寫成下面這樣會報錯。

```javascript
let { x, ...{ y, z } } = o;
// SyntaxError: ... must be followed by an identifier in declaration contexts
```

解構賦值的一個用處，是擴充套件某個函式的引數，引入其他操作。

```javascript
function baseFunction({ a, b }) {
  // ...
}
function wrapperFunction({ x, y, ...restConfig }) {
  // 使用 x 和 y 引數進行操作
  // 其餘引數傳給原始函式
  return baseFunction(restConfig);
}
```

上面程式碼中，原始函式`baseFunction`接受`a`和`b`作為引數，函式`wrapperFunction`在`baseFunction`的基礎上進行了擴充套件，能夠接受多餘的引數，並且保留原始函式的行為。

### 擴充套件運算子

物件的擴充套件運算子（`...`）用於取出引數物件的所有可遍歷屬性，複製到當前物件之中。

```javascript
let z = { a: 3, b: 4 };
let n = { ...z };
n // { a: 3, b: 4 }
```

由於陣列是特殊的物件，所以物件的擴充套件運算子也可以用於陣列。

```javascript
let foo = { ...['a', 'b', 'c'] };
foo
// {0: "a", 1: "b", 2: "c"}
```

如果擴充套件運算子後面是一個空物件，則沒有任何效果。

```javascript
{...{}, a: 1}
// { a: 1 }
```

如果擴充套件運算子後面不是物件，則會自動將其轉為物件。

```javascript
// 等同於 {...Object(1)}
{...1} // {}
```

上面程式碼中，擴充套件運算子後面是整數`1`，會自動轉為數值的包裝物件`Number{1}`。由於該物件沒有自身屬性，所以返回一個空物件。

下面的例子都是類似的道理。

```javascript
// 等同於 {...Object(true)}
{...true} // {}

// 等同於 {...Object(undefined)}
{...undefined} // {}

// 等同於 {...Object(null)}
{...null} // {}
```

但是，如果擴充套件運算子後面是字串，它會自動轉成一個類似陣列的物件，因此返回的不是空物件。

```javascript
{...'hello'}
// {0: "h", 1: "e", 2: "l", 3: "l", 4: "o"}
```

物件的擴充套件運算子等同於使用`Object.assign()`方法。

```javascript
let aClone = { ...a };
// 等同於
let aClone = Object.assign({}, a);
```

上面的例子只是複製了物件例項的屬性，如果想完整克隆一個物件，還複製物件原型的屬性，可以採用下面的寫法。

```javascript
// 寫法一
const clone1 = {
  __proto__: Object.getPrototypeOf(obj),
  ...obj
};

// 寫法二
const clone2 = Object.assign(
  Object.create(Object.getPrototypeOf(obj)),
  obj
);

// 寫法三
const clone3 = Object.create(
  Object.getPrototypeOf(obj),
  Object.getOwnPropertyDescriptors(obj)
)
```

上面程式碼中，寫法一的`__proto__`屬性在非瀏覽器的環境不一定部署，因此推薦使用寫法二和寫法三。

擴充套件運算子可以用於合併兩個物件。

```javascript
let ab = { ...a, ...b };
// 等同於
let ab = Object.assign({}, a, b);
```

如果使用者自定義的屬性，放在擴充套件運算子後面，則擴充套件運算子內部的同名屬性會被覆蓋掉。

```javascript
let aWithOverrides = { ...a, x: 1, y: 2 };
// 等同於
let aWithOverrides = { ...a, ...{ x: 1, y: 2 } };
// 等同於
let x = 1, y = 2, aWithOverrides = { ...a, x, y };
// 等同於
let aWithOverrides = Object.assign({}, a, { x: 1, y: 2 });
```

上面程式碼中，`a`物件的`x`屬性和`y`屬性，複製到新物件後會被覆蓋掉。

這用來修改現有物件部分的屬性就很方便了。

```javascript
let newVersion = {
  ...previousVersion,
  name: 'New Name' // Override the name property
};
```

上面程式碼中，`newVersion`物件自定義了`name`屬性，其他屬性全部複製自`previousVersion`物件。

如果把自定義屬性放在擴充套件運算子前面，就變成了設定新物件的預設屬性值。

```javascript
let aWithDefaults = { x: 1, y: 2, ...a };
// 等同於
let aWithDefaults = Object.assign({}, { x: 1, y: 2 }, a);
// 等同於
let aWithDefaults = Object.assign({ x: 1, y: 2 }, a);
```

與陣列的擴充套件運算子一樣，物件的擴充套件運算子後面可以跟表示式。

```javascript
const obj = {
  ...(x > 1 ? {a: 1} : {}),
  b: 2,
};
```

擴充套件運算子的引數物件之中，如果有取值函式`get`，這個函式是會執行的。

```javascript
let a = {
  get x() {
    throw new Error('not throw yet');
  }
}

let aWithXGetter = { ...a }; // 報錯
```

上面例子中，取值函式`get`在擴充套件`a`物件時會自動執行，導致報錯。

## 鏈判斷運算子

程式設計實務中，如果讀取物件內部的某個屬性，往往需要判斷一下該物件是否存在。比如，要讀取`message.body.user.firstName`，安全的寫法是寫成下面這樣。

```javascript
// 錯誤的寫法
const  firstName = message.body.user.firstName;

// 正確的寫法
const firstName = (message
  && message.body
  && message.body.user
  && message.body.user.firstName) || 'default';
```

上面例子中，`firstName`屬性在物件的第四層，所以需要判斷四次，每一層是否有值。

三元運算子`?:`也常用於判斷物件是否存在。

```javascript
const fooInput = myForm.querySelector('input[name=foo]')
const fooValue = fooInput ? fooInput.value : undefined
```

上面例子中，必須先判斷`fooInput`是否存在，才能讀取`fooInput.value`。

這樣的層層判斷非常麻煩，因此 [ES2020](https://github.com/tc39/proposal-optional-chaining) 引入了“鏈判斷運算子”（optional chaining operator）`?.`，簡化上面的寫法。

```javascript
const firstName = message?.body?.user?.firstName || 'default';
const fooValue = myForm.querySelector('input[name=foo]')?.value
```

上面程式碼使用了`?.`運算子，直接在鏈式呼叫的時候判斷，左側的物件是否為`null`或`undefined`。如果是的，就不再往下運算，而是返回`undefined`。

下面是判斷物件方法是否存在，如果存在就立即執行的例子。

```javascript
iterator.return?.()
```

上面程式碼中，`iterator.return`如果有定義，就會呼叫該方法，否則`iterator.return`直接返回`undefined`，不再執行`?.`後面的部分。

對於那些可能沒有實現的方法，這個運算子尤其有用。

```javascript
if (myForm.checkValidity?.() === false) {
  // 表單校驗失敗
  return;
}
```

上面程式碼中，老式瀏覽器的表單可能沒有`checkValidity`這個方法，這時`?.`運算子就會返回`undefined`，判斷語句就變成了`undefined === false`，所以就會跳過下面的程式碼。

鏈判斷運算子有三種用法。

- `obj?.prop` // 物件屬性
- `obj?.[expr]` // 同上
- `func?.(...args)` // 函式或物件方法的呼叫

下面是`obj?.[expr]`用法的一個例子。

```bash
let hex = "#C0FFEE".match(/#([A-Z]+)/i)?.[1];
```

上面例子中，字串的`match()`方法，如果沒有發現匹配會返回`null`，如果發現匹配會返回一個數組，`?.`運算子起到了判斷作用。

下面是`?.`運算子常見形式，以及不使用該運算子時的等價形式。

```javascript
a?.b
// 等同於
a == null ? undefined : a.b

a?.[x]
// 等同於
a == null ? undefined : a[x]

a?.b()
// 等同於
a == null ? undefined : a.b()

a?.()
// 等同於
a == null ? undefined : a()
```

上面程式碼中，特別注意後兩種形式，如果`a?.b()`裡面的`a.b`不是函式，不可呼叫，那麼`a?.b()`是會報錯的。`a?.()`也是如此，如果`a`不是`null`或`undefined`，但也不是函式，那麼`a?.()`會報錯。

使用這個運算子，有幾個注意點。

（1）短路機制

`?.`運算子相當於一種短路機制，只要不滿足條件，就不再往下執行。

```javascript
a?.[++x]
// 等同於
a == null ? undefined : a[++x]
```

上面程式碼中，如果`a`是`undefined`或`null`，那麼`x`不會進行遞增運算。也就是說，鏈判斷運算子一旦為真，右側的表示式就不再求值。

（2）delete 運算子

```javascript
delete a?.b
// 等同於
a == null ? undefined : delete a.b
```

上面程式碼中，如果`a`是`undefined`或`null`，會直接返回`undefined`，而不會進行`delete`運算。

（3）括號的影響

如果屬性鏈有圓括號，鏈判斷運算子對圓括號外部沒有影響，只對圓括號內部有影響。

```javascript
(a?.b).c
// 等價於
(a == null ? undefined : a.b).c
```

上面程式碼中，`?.`對圓括號外部沒有影響，不管`a`物件是否存在，圓括號後面的`.c`總是會執行。

一般來說，使用`?.`運算子的場合，不應該使用圓括號。

（4）報錯場合

以下寫法是禁止的，會報錯。

```javascript
// 建構函式
new a?.()
new a?.b()

// 鏈判斷運算子的右側有模板字串
a?.`{b}`
a?.b`{c}`

// 鏈判斷運算子的左側是 super
super?.()
super?.foo

// 鏈運算子用於賦值運算子左側
a?.b = c
```

（5）右側不得為十進位制數值

為了保證相容以前的程式碼，允許`foo?.3:0`被解析成`foo ? .3 : 0`，因此規定如果`?.`後面緊跟一個十進位制數字，那麼`?.`不再被看成是一個完整的運算子，而會按照三元運算子進行處理，也就是說，那個小數點會歸屬於後面的十進位制數字，形成一個小數。

## Null 判斷運算子

讀取物件屬性的時候，如果某個屬性的值是`null`或`undefined`，有時候需要為它們指定預設值。常見做法是透過`||`運算子指定預設值。

```javascript
const headerText = response.settings.headerText || 'Hello, world!';
const animationDuration = response.settings.animationDuration || 300;
const showSplashScreen = response.settings.showSplashScreen || true;
```

上面的三行程式碼都透過`||`運算子指定預設值，但是這樣寫是錯的。開發者的原意是，只要屬性的值為`null`或`undefined`，預設值就會生效，但是屬性的值如果為空字串或`false`或`0`，預設值也會生效。

為了避免這種情況，[ES2020](https://github.com/tc39/proposal-nullish-coalescing) 引入了一個新的 Null 判斷運算子`??`。它的行為類似`||`，但是隻有運算子左側的值為`null`或`undefined`時，才會返回右側的值。

```javascript
const headerText = response.settings.headerText ?? 'Hello, world!';
const animationDuration = response.settings.animationDuration ?? 300;
const showSplashScreen = response.settings.showSplashScreen ?? true;
```

上面程式碼中，預設值只有在左側屬性值為`null`或`undefined`時，才會生效。

這個運算子的一個目的，就是跟鏈判斷運算子`?.`配合使用，為`null`或`undefined`的值設定預設值。

```javascript
const animationDuration = response.settings?.animationDuration ?? 300;
```

上面程式碼中，如果`response.settings`是`null`或`undefined`，或者`response.settings.animationDuration`是`null`或`undefined`，就會返回預設值300。也就是說，這一行程式碼包括了兩級屬性的判斷。

這個運算子很適合判斷函式引數是否賦值。

```javascript
function Component(props) {
  const enable = props.enabled ?? true;
  // …
}
```

上面程式碼判斷`props`引數的`enabled`屬性是否賦值，基本等同於下面的寫法。

```javascript
function Component(props) {
  const {
    enabled: enable = true,
  } = props;
  // …
}
```

`??`有一個運算優先順序問題，它與`&&`和`||`的優先順序孰高孰低。現在的規則是，如果多個邏輯運算子一起使用，必須用括號表明優先順序，否則會報錯。

```javascript
// 報錯
lhs && middle ?? rhs
lhs ?? middle && rhs
lhs || middle ?? rhs
lhs ?? middle || rhs
```

上面四個表示式都會報錯，必須加入表明優先順序的括號。

```javascript
(lhs && middle) ?? rhs;
lhs && (middle ?? rhs);

(lhs ?? middle) && rhs;
lhs ?? (middle && rhs);

(lhs || middle) ?? rhs;
lhs || (middle ?? rhs);

(lhs ?? middle) || rhs;
lhs ?? (middle || rhs);
```

