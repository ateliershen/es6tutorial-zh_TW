# async 函式

## 含義

ES2017 標準引入了 async 函式，使得非同步操作變得更加方便。

async 函式是什麼？一句話，它就是 Generator 函式的語法糖。

前文有一個 Generator 函式，依次讀取兩個檔案。

```javascript
const fs = require('fs');

const readFile = function (fileName) {
  return new Promise(function (resolve, reject) {
    fs.readFile(fileName, function(error, data) {
      if (error) return reject(error);
      resolve(data);
    });
  });
};

const gen = function* () {
  const f1 = yield readFile('/etc/fstab');
  const f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

上面程式碼的函式`gen`可以寫成`async`函式，就是下面這樣。

```javascript
const asyncReadFile = async function () {
  const f1 = await readFile('/etc/fstab');
  const f2 = await readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

一比較就會發現，`async`函式就是將 Generator 函式的星號（`*`）替換成`async`，將`yield`替換成`await`，僅此而已。

`async`函式對 Generator 函式的改進，體現在以下四點。

（1）內建執行器。

Generator 函式的執行必須靠執行器，所以才有了`co`模組，而`async`函式自帶執行器。也就是說，`async`函式的執行，與普通函式一模一樣，只要一行。

```javascript
asyncReadFile();
```

上面的程式碼呼叫了`asyncReadFile`函式，然後它就會自動執行，輸出最後結果。這完全不像 Generator 函式，需要呼叫`next`方法，或者用`co`模組，才能真正執行，得到最後結果。

（2）更好的語義。

`async`和`await`，比起星號和`yield`，語義更清楚了。`async`表示函式裡有非同步操作，`await`表示緊跟在後面的表示式需要等待結果。

（3）更廣的適用性。

`co`模組約定，`yield`命令後面只能是 Thunk 函式或 Promise 物件，而`async`函式的`await`命令後面，可以是 Promise 物件和原始型別的值（數值、字串和布林值，但這時會自動轉成立即 resolved 的 Promise 物件）。

（4）返回值是 Promise。

`async`函式的返回值是 Promise 物件，這比 Generator 函式的返回值是 Iterator 物件方便多了。你可以用`then`方法指定下一步的操作。

進一步說，`async`函式完全可以看作多個非同步操作，包裝成的一個 Promise 物件，而`await`命令就是內部`then`命令的語法糖。

## 基本用法

`async`函式返回一個 Promise 物件，可以使用`then`方法添加回調函式。當函式執行的時候，一旦遇到`await`就會先返回，等到非同步操作完成，再接著執行函式體內後面的語句。

下面是一個例子。

```javascript
async function getStockPriceByName(name) {
  const symbol = await getStockSymbol(name);
  const stockPrice = await getStockPrice(symbol);
  return stockPrice;
}

getStockPriceByName('goog').then(function (result) {
  console.log(result);
});
```

上面程式碼是一個獲取股票報價的函式，函式前面的`async`關鍵字，表明該函式內部有非同步操作。呼叫該函式時，會立即返回一個`Promise`物件。

下面是另一個例子，指定多少毫秒後輸出一個值。

```javascript
function timeout(ms) {
  return new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}

async function asyncPrint(value, ms) {
  await timeout(ms);
  console.log(value);
}

asyncPrint('hello world', 50);
```

上面程式碼指定 50 毫秒以後，輸出`hello world`。

由於`async`函式返回的是 Promise 物件，可以作為`await`命令的引數。所以，上面的例子也可以寫成下面的形式。

```javascript
async function timeout(ms) {
  await new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}

async function asyncPrint(value, ms) {
  await timeout(ms);
  console.log(value);
}

asyncPrint('hello world', 50);
```

async 函式有多種使用形式。

```javascript
// 函式宣告
async function foo() {}

// 函式表示式
const foo = async function () {};

// 物件的方法
let obj = { async foo() {} };
obj.foo().then(...)

// Class 的方法
class Storage {
  constructor() {
    this.cachePromise = caches.open('avatars');
  }

  async getAvatar(name) {
    const cache = await this.cachePromise;
    return cache.match(`/avatars/${name}.jpg`);
  }
}

const storage = new Storage();
storage.getAvatar('jake').then(…);

// 箭頭函式
const foo = async () => {};
```

## 語法

`async`函式的語法規則總體上比較簡單，難點是錯誤處理機制。

### 返回 Promise 物件

`async`函式返回一個 Promise 物件。

`async`函式內部`return`語句返回的值，會成為`then`方法回撥函式的引數。

```javascript
async function f() {
  return 'hello world';
}

f().then(v => console.log(v))
// "hello world"
```

上面程式碼中，函式`f`內部`return`命令返回的值，會被`then`方法回撥函式接收到。

`async`函式內部丟擲錯誤，會導致返回的 Promise 物件變為`reject`狀態。丟擲的錯誤物件會被`catch`方法回撥函式接收到。

```javascript
async function f() {
  throw new Error('出錯了');
}

f().then(
  v => console.log('resolve', v),
  e => console.log('reject', e)
)
//reject Error: 出錯了
```

### Promise 物件的狀態變化

`async`函式返回的 Promise 物件，必須等到內部所有`await`命令後面的 Promise 物件執行完，才會發生狀態改變，除非遇到`return`語句或者丟擲錯誤。也就是說，只有`async`函式內部的非同步操作執行完，才會執行`then`方法指定的回撥函式。

下面是一個例子。

```javascript
async function getTitle(url) {
  let response = await fetch(url);
  let html = await response.text();
  return html.match(/<title>([\s\S]+)<\/title>/i)[1];
}
getTitle('https://tc39.github.io/ecma262/').then(console.log)
// "ECMAScript 2017 Language Specification"
```

上面程式碼中，函式`getTitle`內部有三個操作：抓取網頁、取出文字、匹配頁面標題。只有這三個操作全部完成，才會執行`then`方法裡面的`console.log`。

### await 命令

正常情況下，`await`命令後面是一個 Promise 物件，返回該物件的結果。如果不是 Promise 物件，就直接返回對應的值。

```javascript
async function f() {
  // 等同於
  // return 123;
  return await 123;
}

f().then(v => console.log(v))
// 123
```

上面程式碼中，`await`命令的引數是數值`123`，這時等同於`return 123`。

另一種情況是，`await`命令後面是一個`thenable`物件（即定義了`then`方法的物件），那麼`await`會將其等同於 Promise 物件。

```javascript
class Sleep {
  constructor(timeout) {
    this.timeout = timeout;
  }
  then(resolve, reject) {
    const startTime = Date.now();
    setTimeout(
      () => resolve(Date.now() - startTime),
      this.timeout
    );
  }
}

(async () => {
  const sleepTime = await new Sleep(1000);
  console.log(sleepTime);
})();
// 1000
```

上面程式碼中，`await`命令後面是一個`Sleep`物件的例項。這個例項不是 Promise 物件，但是因為定義了`then`方法，`await`會將其視為`Promise`處理。

這個例子還演示瞭如何實現休眠效果。JavaScript 一直沒有休眠的語法，但是藉助`await`命令就可以讓程式停頓指定的時間。下面給出了一個簡化的`sleep`實現。

```javascript
function sleep(interval) {
  return new Promise(resolve => {
    setTimeout(resolve, interval);
  })
}

// 用法
async function one2FiveInAsync() {
  for(let i = 1; i <= 5; i++) {
    console.log(i);
    await sleep(1000);
  }
}

one2FiveInAsync();
```

`await`命令後面的 Promise 物件如果變為`reject`狀態，則`reject`的引數會被`catch`方法的回撥函式接收到。

```javascript
async function f() {
  await Promise.reject('出錯了');
}

f()
.then(v => console.log(v))
.catch(e => console.log(e))
// 出錯了
```

注意，上面程式碼中，`await`語句前面沒有`return`，但是`reject`方法的引數依然傳入了`catch`方法的回撥函式。這裡如果在`await`前面加上`return`，效果是一樣的。

任何一個`await`語句後面的 Promise 物件變為`reject`狀態，那麼整個`async`函式都會中斷執行。

```javascript
async function f() {
  await Promise.reject('出錯了');
  await Promise.resolve('hello world'); // 不會執行
}
```

上面程式碼中，第二個`await`語句是不會執行的，因為第一個`await`語句狀態變成了`reject`。

有時，我們希望即使前一個非同步操作失敗，也不要中斷後面的非同步操作。這時可以將第一個`await`放在`try...catch`結構裡面，這樣不管這個非同步操作是否成功，第二個`await`都會執行。

```javascript
async function f() {
  try {
    await Promise.reject('出錯了');
  } catch(e) {
  }
  return await Promise.resolve('hello world');
}

f()
.then(v => console.log(v))
// hello world
```

另一種方法是`await`後面的 Promise 物件再跟一個`catch`方法，處理前面可能出現的錯誤。

```javascript
async function f() {
  await Promise.reject('出錯了')
    .catch(e => console.log(e));
  return await Promise.resolve('hello world');
}

f()
.then(v => console.log(v))
// 出錯了
// hello world
```

### 錯誤處理

如果`await`後面的非同步操作出錯，那麼等同於`async`函式返回的 Promise 物件被`reject`。

```javascript
async function f() {
  await new Promise(function (resolve, reject) {
    throw new Error('出錯了');
  });
}

f()
.then(v => console.log(v))
.catch(e => console.log(e))
// Error：出錯了
```

上面程式碼中，`async`函式`f`執行後，`await`後面的 Promise 物件會丟擲一個錯誤物件，導致`catch`方法的回撥函式被呼叫，它的引數就是丟擲的錯誤物件。具體的執行機制，可以參考後文的“async 函式的實現原理”。

防止出錯的方法，也是將其放在`try...catch`程式碼塊之中。

```javascript
async function f() {
  try {
    await new Promise(function (resolve, reject) {
      throw new Error('出錯了');
    });
  } catch(e) {
  }
  return await('hello world');
}
```

如果有多個`await`命令，可以統一放在`try...catch`結構中。

```javascript
async function main() {
  try {
    const val1 = await firstStep();
    const val2 = await secondStep(val1);
    const val3 = await thirdStep(val1, val2);

    console.log('Final: ', val3);
  }
  catch (err) {
    console.error(err);
  }
}
```

下面的例子使用`try...catch`結構，實現多次重複嘗試。

```javascript
const superagent = require('superagent');
const NUM_RETRIES = 3;

async function test() {
  let i;
  for (i = 0; i < NUM_RETRIES; ++i) {
    try {
      await superagent.get('http://google.com/this-throws-an-error');
      break;
    } catch(err) {}
  }
  console.log(i); // 3
}

test();
```

上面程式碼中，如果`await`操作成功，就會使用`break`語句退出迴圈；如果失敗，會被`catch`語句捕捉，然後進入下一輪迴圈。

### 使用注意點

第一點，前面已經說過，`await`命令後面的`Promise`物件，執行結果可能是`rejected`，所以最好把`await`命令放在`try...catch`程式碼塊中。

```javascript
async function myFunction() {
  try {
    await somethingThatReturnsAPromise();
  } catch (err) {
    console.log(err);
  }
}

// 另一種寫法

async function myFunction() {
  await somethingThatReturnsAPromise()
  .catch(function (err) {
    console.log(err);
  });
}
```

第二點，多個`await`命令後面的非同步操作，如果不存在繼發關係，最好讓它們同時觸發。

```javascript
let foo = await getFoo();
let bar = await getBar();
```

上面程式碼中，`getFoo`和`getBar`是兩個獨立的非同步操作（即互不依賴），被寫成繼發關係。這樣比較耗時，因為只有`getFoo`完成以後，才會執行`getBar`，完全可以讓它們同時觸發。

```javascript
// 寫法一
let [foo, bar] = await Promise.all([getFoo(), getBar()]);

// 寫法二
let fooPromise = getFoo();
let barPromise = getBar();
let foo = await fooPromise;
let bar = await barPromise;
```

上面兩種寫法，`getFoo`和`getBar`都是同時觸發，這樣就會縮短程式的執行時間。

第三點，`await`命令只能用在`async`函式之中，如果用在普通函式，就會報錯。

```javascript
async function dbFuc(db) {
  let docs = [{}, {}, {}];

  // 報錯
  docs.forEach(function (doc) {
    await db.post(doc);
  });
}
```

上面程式碼會報錯，因為`await`用在普通函式之中了。但是，如果將`forEach`方法的引數改成`async`函式，也有問題。

```javascript
function dbFuc(db) { //這裡不需要 async
  let docs = [{}, {}, {}];

  // 可能得到錯誤結果
  docs.forEach(async function (doc) {
    await db.post(doc);
  });
}
```

上面程式碼可能不會正常工作，原因是這時三個`db.post()`操作將是併發執行，也就是同時執行，而不是繼發執行。正確的寫法是採用`for`迴圈。

```javascript
async function dbFuc(db) {
  let docs = [{}, {}, {}];

  for (let doc of docs) {
    await db.post(doc);
  }
}
```

另一種方法是使用陣列的`reduce()`方法。

```javascript
async function dbFuc(db) {
  let docs = [{}, {}, {}];

  await docs.reduce(async (_, doc) => {
    await _;
    await db.post(doc);
  }, undefined);
}
```

上面例子中，`reduce()`方法的第一個引數是`async`函式，導致該函式的第一個引數是前一步操作返回的 Promise 物件，所以必須使用`await`等待它操作結束。另外，`reduce()`方法返回的是`docs`陣列最後一個成員的`async`函式的執行結果，也是一個 Promise 物件，導致在它前面也必須加上`await`。

上面的`reduce()`的引數函式裡面沒有`return`語句，原因是這個函式的主要目的是`db.post()`操作，不是返回值。而且`async`函式不管有沒有`return`語句，總是返回一個 Promise 物件，所以這裡的`return`是不必要的。

如果確實希望多個請求併發執行，可以使用`Promise.all`方法。當三個請求都會`resolved`時，下面兩種寫法效果相同。

```javascript
async function dbFuc(db) {
  let docs = [{}, {}, {}];
  let promises = docs.map((doc) => db.post(doc));

  let results = await Promise.all(promises);
  console.log(results);
}

// 或者使用下面的寫法

async function dbFuc(db) {
  let docs = [{}, {}, {}];
  let promises = docs.map((doc) => db.post(doc));

  let results = [];
  for (let promise of promises) {
    results.push(await promise);
  }
  console.log(results);
}
```

第四點，async 函式可以保留執行堆疊。

```javascript
const a = () => {
  b().then(() => c());
};
```

上面程式碼中，函式`a`內部運行了一個非同步任務`b()`。當`b()`執行的時候，函式`a()`不會中斷，而是繼續執行。等到`b()`執行結束，可能`a()`早就執行結束了，`b()`所在的上下文環境已經消失了。如果`b()`或`c()`報錯，錯誤堆疊將不包括`a()`。

現在將這個例子改成`async`函式。

```javascript
const a = async () => {
  await b();
  c();
};
```

上面程式碼中，`b()`執行的時候，`a()`是暫停執行，上下文環境都儲存著。一旦`b()`或`c()`報錯，錯誤堆疊將包括`a()`。

## async 函式的實現原理

async 函式的實現原理，就是將 Generator 函式和自動執行器，包裝在一個函式裡。

```javascript
async function fn(args) {
  // ...
}

// 等同於

function fn(args) {
  return spawn(function* () {
    // ...
  });
}
```

所有的`async`函式都可以寫成上面的第二種形式，其中的`spawn`函式就是自動執行器。

下面給出`spawn`函式的實現，基本就是前文自動執行器的翻版。

```javascript
function spawn(genF) {
  return new Promise(function(resolve, reject) {
    const gen = genF();
    function step(nextF) {
      let next;
      try {
        next = nextF();
      } catch(e) {
        return reject(e);
      }
      if(next.done) {
        return resolve(next.value);
      }
      Promise.resolve(next.value).then(function(v) {
        step(function() { return gen.next(v); });
      }, function(e) {
        step(function() { return gen.throw(e); });
      });
    }
    step(function() { return gen.next(undefined); });
  });
}
```

## 與其他非同步處理方法的比較

我們透過一個例子，來看 async 函式與 Promise、Generator 函式的比較。

假定某個 DOM 元素上面，部署了一系列的動畫，前一個動畫結束，才能開始後一個。如果當中有一個動畫出錯，就不再往下執行，返回上一個成功執行的動畫的返回值。

首先是 Promise 的寫法。

```javascript
function chainAnimationsPromise(elem, animations) {

  // 變數ret用來儲存上一個動畫的返回值
  let ret = null;

  // 新建一個空的Promise
  let p = Promise.resolve();

  // 使用then方法，新增所有動畫
  for(let anim of animations) {
    p = p.then(function(val) {
      ret = val;
      return anim(elem);
    });
  }

  // 返回一個部署了錯誤捕捉機制的Promise
  return p.catch(function(e) {
    /* 忽略錯誤，繼續執行 */
  }).then(function() {
    return ret;
  });

}
```

雖然 Promise 的寫法比回撥函式的寫法大大改進，但是一眼看上去，程式碼完全都是 Promise 的 API（`then`、`catch`等等），操作本身的語義反而不容易看出來。

接著是 Generator 函式的寫法。

```javascript
function chainAnimationsGenerator(elem, animations) {

  return spawn(function*() {
    let ret = null;
    try {
      for(let anim of animations) {
        ret = yield anim(elem);
      }
    } catch(e) {
      /* 忽略錯誤，繼續執行 */
    }
    return ret;
  });

}
```

上面程式碼使用 Generator 函式遍歷了每個動畫，語義比 Promise 寫法更清晰，使用者定義的操作全部都出現在`spawn`函式的內部。這個寫法的問題在於，必須有一個任務執行器，自動執行 Generator 函式，上面程式碼的`spawn`函式就是自動執行器，它返回一個 Promise 物件，而且必須保證`yield`語句後面的表示式，必須返回一個 Promise。

最後是 async 函式的寫法。

```javascript
async function chainAnimationsAsync(elem, animations) {
  let ret = null;
  try {
    for(let anim of animations) {
      ret = await anim(elem);
    }
  } catch(e) {
    /* 忽略錯誤，繼續執行 */
  }
  return ret;
}
```

可以看到 Async 函式的實現最簡潔，最符合語義，幾乎沒有語義不相關的程式碼。它將 Generator 寫法中的自動執行器，改在語言層面提供，不暴露給使用者，因此程式碼量最少。如果使用 Generator 寫法，自動執行器需要使用者自己提供。

## 例項：按順序完成非同步操作

實際開發中，經常遇到一組非同步操作，需要按照順序完成。比如，依次遠端讀取一組 URL，然後按照讀取的順序輸出結果。

Promise 的寫法如下。

```javascript
function logInOrder(urls) {
  // 遠端讀取所有URL
  const textPromises = urls.map(url => {
    return fetch(url).then(response => response.text());
  });

  // 按次序輸出
  textPromises.reduce((chain, textPromise) => {
    return chain.then(() => textPromise)
      .then(text => console.log(text));
  }, Promise.resolve());
}
```

上面程式碼使用`fetch`方法，同時遠端讀取一組 URL。每個`fetch`操作都返回一個 Promise 物件，放入`textPromises`陣列。然後，`reduce`方法依次處理每個 Promise 物件，然後使用`then`，將所有 Promise 物件連起來，因此就可以依次輸出結果。

這種寫法不太直觀，可讀性比較差。下面是 async 函式實現。

```javascript
async function logInOrder(urls) {
  for (const url of urls) {
    const response = await fetch(url);
    console.log(await response.text());
  }
}
```

上面程式碼確實大大簡化，問題是所有遠端操作都是繼發。只有前一個 URL 返回結果，才會去讀取下一個 URL，這樣做效率很差，非常浪費時間。我們需要的是併發發出遠端請求。

```javascript
async function logInOrder(urls) {
  // 併發讀取遠端URL
  const textPromises = urls.map(async url => {
    const response = await fetch(url);
    return response.text();
  });

  // 按次序輸出
  for (const textPromise of textPromises) {
    console.log(await textPromise);
  }
}
```

上面程式碼中，雖然`map`方法的引數是`async`函式，但它是併發執行的，因為只有`async`函式內部是繼發執行，外部不受影響。後面的`for..of`迴圈內部使用了`await`，因此實現了按順序輸出。

## 頂層 await

根據語法規格，`await`命令只能出現在 async 函式內部，否則都會報錯。

```javascript
// 報錯
const data = await fetch('https://api.example.com');
```

上面程式碼中，`await`命令獨立使用，沒有放在 async 函式裡面，就會報錯。

目前，有一個[語法提案](https://github.com/tc39/proposal-top-level-await)，允許在模組的頂層獨立使用`await`命令，使得上面那行程式碼不會報錯了。這個提案的目的，是借用`await`解決模組非同步載入的問題。

```javascript
// awaiting.js
let output;
async function main() {
  const dynamic = await import(someMission);
  const data = await fetch(url);
  output = someProcess(dynamic.default, data);
}
main();
export { output };
```

上面程式碼中，模組`awaiting.js`的輸出值`output`，取決於非同步操作。我們把非同步操作包裝在一個 async 函式裡面，然後呼叫這個函式，只有等裡面的非同步操作都執行，變數`output`才會有值，否則就返回`undefined`。

上面的程式碼也可以寫成立即執行函式的形式。

```javascript
// awaiting.js
let output;
(async function main() {
  const dynamic = await import(someMission);
  const data = await fetch(url);
  output = someProcess(dynamic.default, data);
})();
export { output };
```

下面是載入這個模組的寫法。

```javascript
// usage.js
import { output } from "./awaiting.js";

function outputPlusValue(value) { return output + value }

console.log(outputPlusValue(100));
setTimeout(() => console.log(outputPlusValue(100), 1000);
```

上面程式碼中，`outputPlusValue()`的執行結果，完全取決於執行的時間。如果`awaiting.js`裡面的非同步操作沒執行完，載入進來的`output`的值就是`undefined`。

目前的解決方法，就是讓原始模組輸出一個 Promise 物件，從這個 Promise 物件判斷非同步操作有沒有結束。

```javascript
// awaiting.js
let output;
export default (async function main() {
  const dynamic = await import(someMission);
  const data = await fetch(url);
  output = someProcess(dynamic.default, data);
})();
export { output };
```

上面程式碼中，`awaiting.js`除了輸出`output`，還預設輸出一個 Promise 物件（async 函式立即執行後，返回一個 Promise 物件），從這個物件判斷非同步操作是否結束。

下面是載入這個模組的新的寫法。

```javascript
// usage.js
import promise, { output } from "./awaiting.js";

function outputPlusValue(value) { return output + value }

promise.then(() => {
  console.log(outputPlusValue(100));
  setTimeout(() => console.log(outputPlusValue(100), 1000);
});
```

上面程式碼中，將`awaiting.js`物件的輸出，放在`promise.then()`裡面，這樣就能保證非同步操作完成以後，才去讀取`output`。

這種寫法比較麻煩，等於要求模組的使用者遵守一個額外的使用協議，按照特殊的方法使用這個模組。一旦你忘了要用 Promise 載入，只使用正常的載入方法，依賴這個模組的程式碼就可能出錯。而且，如果上面的`usage.js`又有對外的輸出，等於這個依賴鏈的所有模組都要使用 Promise 載入。

頂層的`await`命令，就是為了解決這個問題。它保證只有非同步操作完成，模組才會輸出值。

```javascript
// awaiting.js
const dynamic = import(someMission);
const data = fetch(url);
export const output = someProcess((await dynamic).default, await data);
```

上面程式碼中，兩個非同步操作在輸出的時候，都加上了`await`命令。只有等到非同步操作完成，這個模組才會輸出值。

載入這個模組的寫法如下。

```javascript
// usage.js
import { output } from "./awaiting.js";
function outputPlusValue(value) { return output + value }

console.log(outputPlusValue(100));
setTimeout(() => console.log(outputPlusValue(100), 1000);
```

上面程式碼的寫法，與普通的模組載入完全一樣。也就是說，模組的使用者完全不用關心，依賴模組的內部有沒有非同步操作，正常載入即可。

這時，模組的載入會等待依賴模組（上例是`awaiting.js`）的非同步操作完成，才執行後面的程式碼，有點像暫停在那裡。所以，它總是會得到正確的`output`，不會因為載入時機的不同，而得到不一樣的值。

注意，頂層`await`只能用在 ES6 模組，不能用在 CommonJS 模組。這是因為 CommonJS 模組的`require()`是同步載入，如果有頂層`await`，就沒法處理載入了。

下面是頂層`await`的一些使用場景。

```javascript
// import() 方法載入
const strings = await import(`/i18n/${navigator.language}`);

// 資料庫操作
const connection = await dbConnector();

// 依賴回滾
let jQuery;
try {
  jQuery = await import('https://cdn-a.com/jQuery');
} catch {
  jQuery = await import('https://cdn-b.com/jQuery');
}
```

注意，如果載入多個包含頂層`await`命令的模組，載入命令是同步執行的。

```javascript
// x.js
console.log("X1");
await new Promise(r => setTimeout(r, 1000));
console.log("X2");

// y.js
console.log("Y");

// z.js
import "./x.js";
import "./y.js";
console.log("Z");
```

上面程式碼有三個模組，最後的`z.js`載入`x.js`和`y.js`，列印結果是`X1`、`Y`、`X2`、`Z`。這說明，`z.js`並沒有等待`x.js`載入完成，再去載入`y.js`。

頂層的`await`命令有點像，交出程式碼的執行權給其他的模組載入，等非同步操作完成後，再拿回執行權，繼續向下執行。

