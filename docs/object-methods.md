# 物件的新增方法

本章介紹 Object 物件的新增方法。

## Object.is()

ES5 比較兩個值是否相等，只有兩個運算子：相等運算子（`==`）和嚴格相等運算子（`===`）。它們都有缺點，前者會自動轉換資料型別，後者的`NaN`不等於自身，以及`+0`等於`-0`。JavaScript 缺乏一種運算，在所有環境中，只要兩個值是一樣的，它們就應該相等。

ES6 提出“Same-value equality”（同值相等）演算法，用來解決這個問題。`Object.is`就是部署這個演算法的新方法。它用來比較兩個值是否嚴格相等，與嚴格比較運算子（===）的行為基本一致。

```javascript
Object.is('foo', 'foo')
// true
Object.is({}, {})
// false
```

不同之處只有兩個：一是`+0`不等於`-0`，二是`NaN`等於自身。

```javascript
+0 === -0 //true
NaN === NaN // false

Object.is(+0, -0) // false
Object.is(NaN, NaN) // true
```

ES5 可以透過下面的程式碼，部署`Object.is`。

```javascript
Object.defineProperty(Object, 'is', {
  value: function(x, y) {
    if (x === y) {
      // 針對+0 不等於 -0的情況
      return x !== 0 || 1 / x === 1 / y;
    }
    // 針對NaN的情況
    return x !== x && y !== y;
  },
  configurable: true,
  enumerable: false,
  writable: true
});
```

## Object.assign()

### 基本用法

`Object.assign()`方法用於物件的合併，將源物件（source）的所有可列舉屬性，複製到目標物件（target）。

```javascript
const target = { a: 1 };

const source1 = { b: 2 };
const source2 = { c: 3 };

Object.assign(target, source1, source2);
target // {a:1, b:2, c:3}
```

`Object.assign()`方法的第一個引數是目標物件，後面的引數都是源物件。

注意，如果目標物件與源物件有同名屬性，或多個源物件有同名屬性，則後面的屬性會覆蓋前面的屬性。

```javascript
const target = { a: 1, b: 1 };

const source1 = { b: 2, c: 2 };
const source2 = { c: 3 };

Object.assign(target, source1, source2);
target // {a:1, b:2, c:3}
```

如果只有一個引數，`Object.assign()`會直接返回該引數。

```javascript
const obj = {a: 1};
Object.assign(obj) === obj // true
```

如果該引數不是物件，則會先轉成物件，然後返回。

```javascript
typeof Object.assign(2) // "object"
```

由於`undefined`和`null`無法轉成物件，所以如果它們作為引數，就會報錯。

```javascript
Object.assign(undefined) // 報錯
Object.assign(null) // 報錯
```

如果非物件引數出現在源物件的位置（即非首引數），那麼處理規則有所不同。首先，這些引數都會轉成物件，如果無法轉成物件，就會跳過。這意味著，如果`undefined`和`null`不在首引數，就不會報錯。

```javascript
let obj = {a: 1};
Object.assign(obj, undefined) === obj // true
Object.assign(obj, null) === obj // true
```

其他型別的值（即數值、字串和布林值）不在首引數，也不會報錯。但是，除了字串會以陣列形式，複製入目標物件，其他值都不會產生效果。

```javascript
const v1 = 'abc';
const v2 = true;
const v3 = 10;

const obj = Object.assign({}, v1, v2, v3);
console.log(obj); // { "0": "a", "1": "b", "2": "c" }
```

上面程式碼中，`v1`、`v2`、`v3`分別是字串、布林值和數值，結果只有字串合入目標物件（以字元陣列的形式），數值和布林值都會被忽略。這是因為只有字串的包裝物件，會產生可列舉屬性。

```javascript
Object(true) // {[[PrimitiveValue]]: true}
Object(10)  //  {[[PrimitiveValue]]: 10}
Object('abc') // {0: "a", 1: "b", 2: "c", length: 3, [[PrimitiveValue]]: "abc"}
```

上面程式碼中，布林值、數值、字串分別轉成對應的包裝物件，可以看到它們的原始值都在包裝物件的內部屬性`[[PrimitiveValue]]`上面，這個屬性是不會被`Object.assign()`複製的。只有字串的包裝物件，會產生可列舉的實義屬性，那些屬性則會被複製。

`Object.assign()`複製的屬性是有限制的，只複製源物件的自身屬性（不複製繼承屬性），也不複製不可列舉的屬性（`enumerable: false`）。

```javascript
Object.assign({b: 'c'},
  Object.defineProperty({}, 'invisible', {
    enumerable: false,
    value: 'hello'
  })
)
// { b: 'c' }
```

上面程式碼中，`Object.assign()`要複製的物件只有一個不可列舉屬性`invisible`，這個屬性並沒有被複製進去。

屬性名為 Symbol 值的屬性，也會被`Object.assign()`複製。

```javascript
Object.assign({ a: 'b' }, { [Symbol('c')]: 'd' })
// { a: 'b', Symbol(c): 'd' }
```

### 注意點

**（1）淺複製**

`Object.assign()`方法實行的是淺複製，而不是深複製。也就是說，如果源物件某個屬性的值是物件，那麼目標物件複製得到的是這個物件的引用。

```javascript
const obj1 = {a: {b: 1}};
const obj2 = Object.assign({}, obj1);

obj1.a.b = 2;
obj2.a.b // 2
```

上面程式碼中，源物件`obj1`的`a`屬性的值是一個物件，`Object.assign()`複製得到的是這個物件的引用。這個物件的任何變化，都會反映到目標物件上面。

**（2）同名屬性的替換**

對於這種巢狀的物件，一旦遇到同名屬性，`Object.assign()`的處理方法是替換，而不是新增。

```javascript
const target = { a: { b: 'c', d: 'e' } }
const source = { a: { b: 'hello' } }
Object.assign(target, source)
// { a: { b: 'hello' } }
```

上面程式碼中，`target`物件的`a`屬性被`source`物件的`a`屬性整個替換掉了，而不會得到`{ a: { b: 'hello', d: 'e' } }`的結果。這通常不是開發者想要的，需要特別小心。

一些函式庫提供`Object.assign()`的定製版本（比如 Lodash 的`_.defaultsDeep()`方法），可以得到深複製的合併。

**（3）陣列的處理**

`Object.assign()`可以用來處理陣列，但是會把陣列視為物件。

```javascript
Object.assign([1, 2, 3], [4, 5])
// [4, 5, 3]
```

上面程式碼中，`Object.assign()`把陣列視為屬性名為 0、1、2 的物件，因此源陣列的 0 號屬性`4`覆蓋了目標陣列的 0 號屬性`1`。

**（4）取值函式的處理**

`Object.assign()`只能進行值的複製，如果要複製的值是一個取值函式，那麼將求值後再複製。

```javascript
const source = {
  get foo() { return 1 }
};
const target = {};

Object.assign(target, source)
// { foo: 1 }
```

上面程式碼中，`source`物件的`foo`屬性是一個取值函式，`Object.assign()`不會複製這個取值函式，只會拿到值以後，將這個值複製過去。

### 常見用途

`Object.assign()`方法有很多用處。

**（1）為物件新增屬性**

```javascript
class Point {
  constructor(x, y) {
    Object.assign(this, {x, y});
  }
}
```

上面方法透過`Object.assign()`方法，將`x`屬性和`y`屬性新增到`Point`類的物件例項。

**（2）為物件新增方法**

```javascript
Object.assign(SomeClass.prototype, {
  someMethod(arg1, arg2) {
    ···
  },
  anotherMethod() {
    ···
  }
});

// 等同於下面的寫法
SomeClass.prototype.someMethod = function (arg1, arg2) {
  ···
};
SomeClass.prototype.anotherMethod = function () {
  ···
};
```

上面程式碼使用了物件屬性的簡潔表示法，直接將兩個函式放在大括號中，再使用`assign()`方法新增到`SomeClass.prototype`之中。

**（3）克隆物件**

```javascript
function clone(origin) {
  return Object.assign({}, origin);
}
```

上面程式碼將原始物件複製到一個空物件，就得到了原始物件的克隆。

不過，採用這種方法克隆，只能克隆原始物件自身的值，不能克隆它繼承的值。如果想要保持繼承鏈，可以採用下面的程式碼。

```javascript
function clone(origin) {
  let originProto = Object.getPrototypeOf(origin);
  return Object.assign(Object.create(originProto), origin);
}
```

**（4）合併多個物件**

將多個物件合併到某個物件。

```javascript
const merge =
  (target, ...sources) => Object.assign(target, ...sources);
```

如果希望合併後返回一個新物件，可以改寫上面函式，對一個空物件合併。

```javascript
const merge =
  (...sources) => Object.assign({}, ...sources);
```

**（5）為屬性指定預設值**

```javascript
const DEFAULTS = {
  logLevel: 0,
  outputFormat: 'html'
};

function processContent(options) {
  options = Object.assign({}, DEFAULTS, options);
  console.log(options);
  // ...
}
```

上面程式碼中，`DEFAULTS`物件是預設值，`options`物件是使用者提供的引數。`Object.assign()`方法將`DEFAULTS`和`options`合併成一個新物件，如果兩者有同名屬性，則`options`的屬性值會覆蓋`DEFAULTS`的屬性值。

注意，由於存在淺複製的問題，`DEFAULTS`物件和`options`物件的所有屬性的值，最好都是簡單型別，不要指向另一個物件。否則，`DEFAULTS`物件的該屬性很可能不起作用。

```javascript
const DEFAULTS = {
  url: {
    host: 'example.com',
    port: 7070
  },
};

processContent({ url: {port: 8000} })
// {
//   url: {port: 8000}
// }
```

上面程式碼的原意是將`url.port`改成 8000，`url.host`不變。實際結果卻是`options.url`覆蓋掉`DEFAULTS.url`，所以`url.host`就不存在了。

## Object.getOwnPropertyDescriptors()

ES5 的`Object.getOwnPropertyDescriptor()`方法會返回某個物件屬性的描述物件（descriptor）。ES2017 引入了`Object.getOwnPropertyDescriptors()`方法，返回指定物件所有自身屬性（非繼承屬性）的描述物件。

```javascript
const obj = {
  foo: 123,
  get bar() { return 'abc' }
};

Object.getOwnPropertyDescriptors(obj)
// { foo:
//    { value: 123,
//      writable: true,
//      enumerable: true,
//      configurable: true },
//   bar:
//    { get: [Function: get bar],
//      set: undefined,
//      enumerable: true,
//      configurable: true } }
```

上面程式碼中，`Object.getOwnPropertyDescriptors()`方法返回一個物件，所有原物件的屬性名都是該物件的屬性名，對應的屬性值就是該屬性的描述物件。

該方法的實現非常容易。

```javascript
function getOwnPropertyDescriptors(obj) {
  const result = {};
  for (let key of Reflect.ownKeys(obj)) {
    result[key] = Object.getOwnPropertyDescriptor(obj, key);
  }
  return result;
}
```

該方法的引入目的，主要是為了解決`Object.assign()`無法正確複製`get`屬性和`set`屬性的問題。

```javascript
const source = {
  set foo(value) {
    console.log(value);
  }
};

const target1 = {};
Object.assign(target1, source);

Object.getOwnPropertyDescriptor(target1, 'foo')
// { value: undefined,
//   writable: true,
//   enumerable: true,
//   configurable: true }
```

上面程式碼中，`source`物件的`foo`屬性的值是一個賦值函式，`Object.assign`方法將這個屬性複製給`target1`物件，結果該屬性的值變成了`undefined`。這是因為`Object.assign`方法總是複製一個屬性的值，而不會複製它背後的賦值方法或取值方法。

這時，`Object.getOwnPropertyDescriptors()`方法配合`Object.defineProperties()`方法，就可以實現正確複製。

```javascript
const source = {
  set foo(value) {
    console.log(value);
  }
};

const target2 = {};
Object.defineProperties(target2, Object.getOwnPropertyDescriptors(source));
Object.getOwnPropertyDescriptor(target2, 'foo')
// { get: undefined,
//   set: [Function: set foo],
//   enumerable: true,
//   configurable: true }
```

上面程式碼中，兩個物件合併的邏輯可以寫成一個函式。

```javascript
const shallowMerge = (target, source) => Object.defineProperties(
  target,
  Object.getOwnPropertyDescriptors(source)
);
```

`Object.getOwnPropertyDescriptors()`方法的另一個用處，是配合`Object.create()`方法，將物件屬性克隆到一個新物件。這屬於淺複製。

```javascript
const clone = Object.create(Object.getPrototypeOf(obj),
  Object.getOwnPropertyDescriptors(obj));

// 或者

const shallowClone = (obj) => Object.create(
  Object.getPrototypeOf(obj),
  Object.getOwnPropertyDescriptors(obj)
);
```

上面程式碼會克隆物件`obj`。

另外，`Object.getOwnPropertyDescriptors()`方法可以實現一個物件繼承另一個物件。以前，繼承另一個物件，常常寫成下面這樣。

```javascript
const obj = {
  __proto__: prot,
  foo: 123,
};
```

ES6 規定`__proto__`只有瀏覽器要部署，其他環境不用部署。如果去除`__proto__`，上面程式碼就要改成下面這樣。

```javascript
const obj = Object.create(prot);
obj.foo = 123;

// 或者

const obj = Object.assign(
  Object.create(prot),
  {
    foo: 123,
  }
);
```

有了`Object.getOwnPropertyDescriptors()`，我們就有了另一種寫法。

```javascript
const obj = Object.create(
  prot,
  Object.getOwnPropertyDescriptors({
    foo: 123,
  })
);
```

`Object.getOwnPropertyDescriptors()`也可以用來實現 Mixin（混入）模式。

```javascript
let mix = (object) => ({
  with: (...mixins) => mixins.reduce(
    (c, mixin) => Object.create(
      c, Object.getOwnPropertyDescriptors(mixin)
    ), object)
});

// multiple mixins example
let a = {a: 'a'};
let b = {b: 'b'};
let c = {c: 'c'};
let d = mix(c).with(a, b);

d.c // "c"
d.b // "b"
d.a // "a"
```

上面程式碼返回一個新的物件`d`，代表了物件`a`和`b`被混入了物件`c`的操作。

出於完整性的考慮，`Object.getOwnPropertyDescriptors()`進入標準以後，以後還會新增`Reflect.getOwnPropertyDescriptors()`方法。

## `__proto__`屬性，Object.setPrototypeOf()，Object.getPrototypeOf()

JavaScript 語言的物件繼承是透過原型鏈實現的。ES6 提供了更多原型物件的操作方法。

### `__proto__`屬性

`__proto__`屬性（前後各兩個下劃線），用來讀取或設定當前物件的原型物件（prototype）。目前，所有瀏覽器（包括 IE11）都部署了這個屬性。

```javascript
// es5 的寫法
const obj = {
  method: function() { ... }
};
obj.__proto__ = someOtherObj;

// es6 的寫法
var obj = Object.create(someOtherObj);
obj.method = function() { ... };
```

該屬性沒有寫入 ES6 的正文，而是寫入了附錄，原因是`__proto__`前後的雙下劃線，說明它本質上是一個內部屬性，而不是一個正式的對外的 API，只是由於瀏覽器廣泛支援，才被加入了 ES6。標準明確規定，只有瀏覽器必須部署這個屬性，其他執行環境不一定需要部署，而且新的程式碼最好認為這個屬性是不存在的。因此，無論從語義的角度，還是從相容性的角度，都不要使用這個屬性，而是使用下面的`Object.setPrototypeOf()`（寫操作）、`Object.getPrototypeOf()`（讀操作）、`Object.create()`（生成操作）代替。

實現上，`__proto__`呼叫的是`Object.prototype.__proto__`，具體實現如下。

```javascript
Object.defineProperty(Object.prototype, '__proto__', {
  get() {
    let _thisObj = Object(this);
    return Object.getPrototypeOf(_thisObj);
  },
  set(proto) {
    if (this === undefined || this === null) {
      throw new TypeError();
    }
    if (!isObject(this)) {
      return undefined;
    }
    if (!isObject(proto)) {
      return undefined;
    }
    let status = Reflect.setPrototypeOf(this, proto);
    if (!status) {
      throw new TypeError();
    }
  },
});

function isObject(value) {
  return Object(value) === value;
}
```

如果一個物件本身部署了`__proto__`屬性，該屬性的值就是物件的原型。

```javascript
Object.getPrototypeOf({ __proto__: null })
// null
```

### Object.setPrototypeOf()

`Object.setPrototypeOf`方法的作用與`__proto__`相同，用來設定一個物件的原型物件（prototype），返回引數物件本身。它是 ES6 正式推薦的設定原型物件的方法。

```javascript
// 格式
Object.setPrototypeOf(object, prototype)

// 用法
const o = Object.setPrototypeOf({}, null);
```

該方法等同於下面的函式。

```javascript
function setPrototypeOf(obj, proto) {
  obj.__proto__ = proto;
  return obj;
}
```

下面是一個例子。

```javascript
let proto = {};
let obj = { x: 10 };
Object.setPrototypeOf(obj, proto);

proto.y = 20;
proto.z = 40;

obj.x // 10
obj.y // 20
obj.z // 40
```

上面程式碼將`proto`物件設為`obj`物件的原型，所以從`obj`物件可以讀取`proto`物件的屬性。

如果第一個引數不是物件，會自動轉為物件。但是由於返回的還是第一個引數，所以這個操作不會產生任何效果。

```javascript
Object.setPrototypeOf(1, {}) === 1 // true
Object.setPrototypeOf('foo', {}) === 'foo' // true
Object.setPrototypeOf(true, {}) === true // true
```

由於`undefined`和`null`無法轉為物件，所以如果第一個引數是`undefined`或`null`，就會報錯。

```javascript
Object.setPrototypeOf(undefined, {})
// TypeError: Object.setPrototypeOf called on null or undefined

Object.setPrototypeOf(null, {})
// TypeError: Object.setPrototypeOf called on null or undefined
```

### Object.getPrototypeOf()

該方法與`Object.setPrototypeOf`方法配套，用於讀取一個物件的原型物件。

```javascript
Object.getPrototypeOf(obj);
```

下面是一個例子。

```javascript
function Rectangle() {
  // ...
}

const rec = new Rectangle();

Object.getPrototypeOf(rec) === Rectangle.prototype
// true

Object.setPrototypeOf(rec, Object.prototype);
Object.getPrototypeOf(rec) === Rectangle.prototype
// false
```

如果引數不是物件，會被自動轉為物件。

```javascript
// 等同於 Object.getPrototypeOf(Number(1))
Object.getPrototypeOf(1)
// Number {[[PrimitiveValue]]: 0}

// 等同於 Object.getPrototypeOf(String('foo'))
Object.getPrototypeOf('foo')
// String {length: 0, [[PrimitiveValue]]: ""}

// 等同於 Object.getPrototypeOf(Boolean(true))
Object.getPrototypeOf(true)
// Boolean {[[PrimitiveValue]]: false}

Object.getPrototypeOf(1) === Number.prototype // true
Object.getPrototypeOf('foo') === String.prototype // true
Object.getPrototypeOf(true) === Boolean.prototype // true
```

如果引數是`undefined`或`null`，它們無法轉為物件，所以會報錯。

```javascript
Object.getPrototypeOf(null)
// TypeError: Cannot convert undefined or null to object

Object.getPrototypeOf(undefined)
// TypeError: Cannot convert undefined or null to object
```

## Object.keys()，Object.values()，Object.entries()

### Object.keys()

ES5 引入了`Object.keys`方法，返回一個數組，成員是引數物件自身的（不含繼承的）所有可遍歷（enumerable）屬性的鍵名。

```javascript
var obj = { foo: 'bar', baz: 42 };
Object.keys(obj)
// ["foo", "baz"]
```

ES2017 [引入](https://github.com/tc39/proposal-object-values-entries)了跟`Object.keys`配套的`Object.values`和`Object.entries`，作為遍歷一個物件的補充手段，供`for...of`迴圈使用。

```javascript
let {keys, values, entries} = Object;
let obj = { a: 1, b: 2, c: 3 };

for (let key of keys(obj)) {
  console.log(key); // 'a', 'b', 'c'
}

for (let value of values(obj)) {
  console.log(value); // 1, 2, 3
}

for (let [key, value] of entries(obj)) {
  console.log([key, value]); // ['a', 1], ['b', 2], ['c', 3]
}
```

### Object.values()

`Object.values`方法返回一個數組，成員是引數物件自身的（不含繼承的）所有可遍歷（enumerable）屬性的鍵值。

```javascript
const obj = { foo: 'bar', baz: 42 };
Object.values(obj)
// ["bar", 42]
```

返回陣列的成員順序，與本章的《屬性的遍歷》部分介紹的排列規則一致。

```javascript
const obj = { 100: 'a', 2: 'b', 7: 'c' };
Object.values(obj)
// ["b", "c", "a"]
```

上面程式碼中，屬性名為數值的屬性，是按照數值大小，從小到大遍歷的，因此返回的順序是`b`、`c`、`a`。

`Object.values`只返回物件自身的可遍歷屬性。

```javascript
const obj = Object.create({}, {p: {value: 42}});
Object.values(obj) // []
```

上面程式碼中，`Object.create`方法的第二個引數新增的物件屬性（屬性`p`），如果不顯式宣告，預設是不可遍歷的，因為`p`的屬性描述物件的`enumerable`預設是`false`，`Object.values`不會返回這個屬性。只要把`enumerable`改成`true`，`Object.values`就會返回屬性`p`的值。

```javascript
const obj = Object.create({}, {p:
  {
    value: 42,
    enumerable: true
  }
});
Object.values(obj) // [42]
```

`Object.values`會過濾屬性名為 Symbol 值的屬性。

```javascript
Object.values({ [Symbol()]: 123, foo: 'abc' });
// ['abc']
```

如果`Object.values`方法的引數是一個字串，會返回各個字元組成的一個數組。

```javascript
Object.values('foo')
// ['f', 'o', 'o']
```

上面程式碼中，字串會先轉成一個類似陣列的物件。字串的每個字元，就是該物件的一個屬性。因此，`Object.values`返回每個屬性的鍵值，就是各個字元組成的一個數組。

如果引數不是物件，`Object.values`會先將其轉為物件。由於數值和布林值的包裝物件，都不會為例項新增非繼承的屬性。所以，`Object.values`會返回空陣列。

```javascript
Object.values(42) // []
Object.values(true) // []
```

### Object.entries()

`Object.entries()`方法返回一個數組，成員是引數物件自身的（不含繼承的）所有可遍歷（enumerable）屬性的鍵值對陣列。

```javascript
const obj = { foo: 'bar', baz: 42 };
Object.entries(obj)
// [ ["foo", "bar"], ["baz", 42] ]
```

除了返回值不一樣，該方法的行為與`Object.values`基本一致。

如果原物件的屬性名是一個 Symbol 值，該屬性會被忽略。

```javascript
Object.entries({ [Symbol()]: 123, foo: 'abc' });
// [ [ 'foo', 'abc' ] ]
```

上面程式碼中，原物件有兩個屬性，`Object.entries`只輸出屬性名非 Symbol 值的屬性。將來可能會有`Reflect.ownEntries()`方法，返回物件自身的所有屬性。

`Object.entries`的基本用途是遍歷物件的屬性。

```javascript
let obj = { one: 1, two: 2 };
for (let [k, v] of Object.entries(obj)) {
  console.log(
    `${JSON.stringify(k)}: ${JSON.stringify(v)}`
  );
}
// "one": 1
// "two": 2
```

`Object.entries`方法的另一個用處是，將物件轉為真正的`Map`結構。

```javascript
const obj = { foo: 'bar', baz: 42 };
const map = new Map(Object.entries(obj));
map // Map { foo: "bar", baz: 42 }
```

自己實現`Object.entries`方法，非常簡單。

```javascript
// Generator函式的版本
function* entries(obj) {
  for (let key of Object.keys(obj)) {
    yield [key, obj[key]];
  }
}

// 非Generator函式的版本
function entries(obj) {
  let arr = [];
  for (let key of Object.keys(obj)) {
    arr.push([key, obj[key]]);
  }
  return arr;
}
```

## Object.fromEntries()

`Object.fromEntries()`方法是`Object.entries()`的逆操作，用於將一個鍵值對陣列轉為物件。

```javascript
Object.fromEntries([
  ['foo', 'bar'],
  ['baz', 42]
])
// { foo: "bar", baz: 42 }
```

該方法的主要目的，是將鍵值對的資料結構還原為物件，因此特別適合將 Map 結構轉為物件。

```javascript
// 例一
const entries = new Map([
  ['foo', 'bar'],
  ['baz', 42]
]);

Object.fromEntries(entries)
// { foo: "bar", baz: 42 }

// 例二
const map = new Map().set('foo', true).set('bar', false);
Object.fromEntries(map)
// { foo: true, bar: false }
```

該方法的一個用處是配合`URLSearchParams`物件，將查詢字串轉為物件。

```javascript
Object.fromEntries(new URLSearchParams('foo=bar&baz=qux'))
// { foo: "bar", baz: "qux" }
```
