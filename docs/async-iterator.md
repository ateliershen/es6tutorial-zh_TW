# 非同步遍歷器

## 同步遍歷器的問題

《遍歷器》一章說過，Iterator 介面是一種資料遍歷的協議，只要呼叫遍歷器物件的`next`方法，就會得到一個物件，表示當前遍歷指標所在的那個位置的資訊。`next`方法返回的物件的結構是`{value, done}`，其中`value`表示當前的資料的值，`done`是一個布林值，表示遍歷是否結束。

```javascript
function idMaker() {
  let index = 0;

  return {
    next: function() {
      return { value: index++, done: false };
    }
  };
}

const it = idMaker();

it.next().value // 0
it.next().value // 1
it.next().value // 2
// ...
```

上面程式碼中，變數`it`是一個遍歷器（iterator）。每次呼叫`it.next()`方法，就返回一個物件，表示當前遍歷位置的資訊。

這裡隱含著一個規定，`it.next()`方法必須是同步的，只要呼叫就必須立刻返回值。也就是說，一旦執行`it.next()`方法，就必須同步地得到`value`和`done`這兩個屬性。如果遍歷指標正好指向同步操作，當然沒有問題，但對於非同步操作，就不太合適了。

```javascript
function idMaker() {
  let index = 0;

  return {
    next: function() {
      return new Promise(function (resolve, reject) {
        setTimeout(() => {
          resolve({ value: index++, done: false });
        }, 1000);
      });
    }
  };
}
```

上面程式碼中，`next()`方法返回的是一個 Promise 物件，這樣就不行，不符合 Iterator 協議，只要程式碼裡面包含非同步操作都不行。也就是說，Iterator 協議裡面`next()`方法只能包含同步操作。

目前的解決方法是，將非同步操作包裝成 Thunk 函式或者 Promise 物件，即`next()`方法返回值的`value`屬性是一個 Thunk 函式或者 Promise 物件，等待以後返回真正的值，而`done`屬性則還是同步產生的。

```javascript
function idMaker() {
  let index = 0;

  return {
    next: function() {
      return {
        value: new Promise(resolve => setTimeout(() => resolve(index++), 1000)),
        done: false
      };
    }
  };
}

const it = idMaker();

it.next().value.then(o => console.log(o)) // 0
it.next().value.then(o => console.log(o)) // 1
it.next().value.then(o => console.log(o)) // 2
// ...
```

上面程式碼中，`value`屬性的返回值是一個 Promise 物件，用來放置非同步操作。但是這樣寫很麻煩，不太符合直覺，語義也比較繞。

ES2018 [引入](https://github.com/tc39/proposal-async-iteration)了“非同步遍歷器”（Async Iterator），為非同步操作提供原生的遍歷器介面，即`value`和`done`這兩個屬性都是非同步產生。

## 非同步遍歷的介面

非同步遍歷器的最大的語法特點，就是呼叫遍歷器的`next`方法，返回的是一個 Promise 物件。

```javascript
asyncIterator
  .next()
  .then(
    ({ value, done }) => /* ... */
  );
```

上面程式碼中，`asyncIterator`是一個非同步遍歷器，呼叫`next`方法以後，返回一個 Promise 物件。因此，可以使用`then`方法指定，這個 Promise 物件的狀態變為`resolve`以後的回撥函式。回撥函式的引數，則是一個具有`value`和`done`兩個屬性的物件，這個跟同步遍歷器是一樣的。

我們知道，一個物件的同步遍歷器的介面，部署在`Symbol.iterator`屬性上面。同樣地，物件的非同步遍歷器介面，部署在`Symbol.asyncIterator`屬性上面。不管是什麼樣的物件，只要它的`Symbol.asyncIterator`屬性有值，就表示應該對它進行非同步遍歷。

下面是一個非同步遍歷器的例子。

```javascript
const asyncIterable = createAsyncIterable(['a', 'b']);
const asyncIterator = asyncIterable[Symbol.asyncIterator]();

asyncIterator
.next()
.then(iterResult1 => {
  console.log(iterResult1); // { value: 'a', done: false }
  return asyncIterator.next();
})
.then(iterResult2 => {
  console.log(iterResult2); // { value: 'b', done: false }
  return asyncIterator.next();
})
.then(iterResult3 => {
  console.log(iterResult3); // { value: undefined, done: true }
});
```

上面程式碼中，非同步遍歷器其實返回了兩次值。第一次呼叫的時候，返回一個 Promise 物件；等到 Promise 物件`resolve`了，再返回一個表示當前資料成員資訊的物件。這就是說，非同步遍歷器與同步遍歷器最終行為是一致的，只是會先返回 Promise 物件，作為中介。

由於非同步遍歷器的`next`方法，返回的是一個 Promise 物件。因此，可以把它放在`await`命令後面。

```javascript
async function f() {
  const asyncIterable = createAsyncIterable(['a', 'b']);
  const asyncIterator = asyncIterable[Symbol.asyncIterator]();
  console.log(await asyncIterator.next());
  // { value: 'a', done: false }
  console.log(await asyncIterator.next());
  // { value: 'b', done: false }
  console.log(await asyncIterator.next());
  // { value: undefined, done: true }
}
```

上面程式碼中，`next`方法用`await`處理以後，就不必使用`then`方法了。整個流程已經很接近同步處理了。

注意，非同步遍歷器的`next`方法是可以連續呼叫的，不必等到上一步產生的 Promise 物件`resolve`以後再呼叫。這種情況下，`next`方法會累積起來，自動按照每一步的順序執行下去。下面是一個例子，把所有的`next`方法放在`Promise.all`方法裡面。

```javascript
const asyncIterable = createAsyncIterable(['a', 'b']);
const asyncIterator = asyncIterable[Symbol.asyncIterator]();
const [{value: v1}, {value: v2}] = await Promise.all([
  asyncIterator.next(), asyncIterator.next()
]);

console.log(v1, v2); // a b
```

另一種用法是一次性呼叫所有的`next`方法，然後`await`最後一步操作。

```javascript
async function runner() {
  const writer = openFile('someFile.txt');
  writer.next('hello');
  writer.next('world');
  await writer.return();
}

runner();
```

## for await...of

前面介紹過，`for...of`迴圈用於遍歷同步的 Iterator 介面。新引入的`for await...of`迴圈，則是用於遍歷非同步的 Iterator 介面。

```javascript
async function f() {
  for await (const x of createAsyncIterable(['a', 'b'])) {
    console.log(x);
  }
}
// a
// b
```

上面程式碼中，`createAsyncIterable()`返回一個擁有非同步遍歷器介面的物件，`for...of`迴圈自動呼叫這個物件的非同步遍歷器的`next`方法，會得到一個 Promise 物件。`await`用來處理這個 Promise 物件，一旦`resolve`，就把得到的值（`x`）傳入`for...of`的迴圈體。

`for await...of`迴圈的一個用途，是部署了 asyncIterable 操作的非同步介面，可以直接放入這個迴圈。

```javascript
let body = '';

async function f() {
  for await(const data of req) body += data;
  const parsed = JSON.parse(body);
  console.log('got', parsed);
}
```

上面程式碼中，`req`是一個 asyncIterable 物件，用來非同步讀取資料。可以看到，使用`for await...of`迴圈以後，程式碼會非常簡潔。

如果`next`方法返回的 Promise 物件被`reject`，`for await...of`就會報錯，要用`try...catch`捕捉。

```javascript
async function () {
  try {
    for await (const x of createRejectingIterable()) {
      console.log(x);
    }
  } catch (e) {
    console.error(e);
  }
}
```

注意，`for await...of`迴圈也可以用於同步遍歷器。

```javascript
(async function () {
  for await (const x of ['a', 'b']) {
    console.log(x);
  }
})();
// a
// b
```

Node v10 支援非同步遍歷器，Stream 就部署了這個介面。下面是讀取檔案的傳統寫法與非同步遍歷器寫法的差異。

```javascript
// 傳統寫法
function main(inputFilePath) {
  const readStream = fs.createReadStream(
    inputFilePath,
    { encoding: 'utf8', highWaterMark: 1024 }
  );
  readStream.on('data', (chunk) => {
    console.log('>>> '+chunk);
  });
  readStream.on('end', () => {
    console.log('### DONE ###');
  });
}

// 非同步遍歷器寫法
async function main(inputFilePath) {
  const readStream = fs.createReadStream(
    inputFilePath,
    { encoding: 'utf8', highWaterMark: 1024 }
  );

  for await (const chunk of readStream) {
    console.log('>>> '+chunk);
  }
  console.log('### DONE ###');
}
```

## 非同步 Generator 函式

就像 Generator 函式返回一個同步遍歷器物件一樣，非同步 Generator 函式的作用，是返回一個非同步遍歷器物件。

在語法上，非同步 Generator 函式就是`async`函式與 Generator 函式的結合。

```javascript
async function* gen() {
  yield 'hello';
}
const genObj = gen();
genObj.next().then(x => console.log(x));
// { value: 'hello', done: false }
```

上面程式碼中，`gen`是一個非同步 Generator 函式，執行後返回一個非同步 Iterator 物件。對該物件呼叫`next`方法，返回一個 Promise 物件。

非同步遍歷器的設計目的之一，就是 Generator 函式處理同步操作和非同步操作時，能夠使用同一套介面。

```javascript
// 同步 Generator 函式
function* map(iterable, func) {
  const iter = iterable[Symbol.iterator]();
  while (true) {
    const {value, done} = iter.next();
    if (done) break;
    yield func(value);
  }
}

// 非同步 Generator 函式
async function* map(iterable, func) {
  const iter = iterable[Symbol.asyncIterator]();
  while (true) {
    const {value, done} = await iter.next();
    if (done) break;
    yield func(value);
  }
}
```

上面程式碼中，`map`是一個 Generator 函式，第一個引數是可遍歷物件`iterable`，第二個引數是一個回撥函式`func`。`map`的作用是將`iterable`每一步返回的值，使用`func`進行處理。上面有兩個版本的`map`，前一個處理同步遍歷器，後一個處理非同步遍歷器，可以看到兩個版本的寫法基本上是一致的。

下面是另一個非同步 Generator 函式的例子。

```javascript
async function* readLines(path) {
  let file = await fileOpen(path);

  try {
    while (!file.EOF) {
      yield await file.readLine();
    }
  } finally {
    await file.close();
  }
}
```

上面程式碼中，非同步操作前面使用`await`關鍵字標明，即`await`後面的操作，應該返回 Promise 物件。凡是使用`yield`關鍵字的地方，就是`next`方法停下來的地方，它後面的表示式的值（即`await file.readLine()`的值），會作為`next()`返回物件的`value`屬性，這一點是與同步 Generator 函式一致的。

非同步 Generator 函式內部，能夠同時使用`await`和`yield`命令。可以這樣理解，`await`命令用於將外部操作產生的值輸入函式內部，`yield`命令用於將函式內部的值輸出。

上面程式碼定義的非同步 Generator 函式的用法如下。

```javascript
(async function () {
  for await (const line of readLines(filePath)) {
    console.log(line);
  }
})()
```

非同步 Generator 函式可以與`for await...of`迴圈結合起來使用。

```javascript
async function* prefixLines(asyncIterable) {
  for await (const line of asyncIterable) {
    yield '> ' + line;
  }
}
```

非同步 Generator 函式的返回值是一個非同步 Iterator，即每次呼叫它的`next`方法，會返回一個 Promise 物件，也就是說，跟在`yield`命令後面的，應該是一個 Promise 物件。如果像上面那個例子那樣，`yield`命令後面是一個字串，會被自動包裝成一個 Promise 物件。

```javascript
function fetchRandom() {
  const url = 'https://www.random.org/decimal-fractions/'
    + '?num=1&dec=10&col=1&format=plain&rnd=new';
  return fetch(url);
}

async function* asyncGenerator() {
  console.log('Start');
  const result = await fetchRandom(); // (A)
  yield 'Result: ' + await result.text(); // (B)
  console.log('Done');
}

const ag = asyncGenerator();
ag.next().then(({value, done}) => {
  console.log(value);
})
```

上面程式碼中，`ag`是`asyncGenerator`函式返回的非同步遍歷器物件。呼叫`ag.next()`以後，上面程式碼的執行順序如下。

1. `ag.next()`立刻返回一個 Promise 物件。
1. `asyncGenerator`函式開始執行，打印出`Start`。
1. `await`命令返回一個 Promise 物件，`asyncGenerator`函式停在這裡。
1. A 處變成 fulfilled 狀態，產生的值放入`result`變數，`asyncGenerator`函式繼續往下執行。
1. 函式在 B 處的`yield`暫停執行，一旦`yield`命令取到值，`ag.next()`返回的那個 Promise 物件變成 fulfilled 狀態。
1. `ag.next()`後面的`then`方法指定的回撥函式開始執行。該回調函式的引數是一個物件`{value, done}`，其中`value`的值是`yield`命令後面的那個表示式的值，`done`的值是`false`。

A 和 B 兩行的作用類似於下面的程式碼。

```javascript
return new Promise((resolve, reject) => {
  fetchRandom()
  .then(result => result.text())
  .then(result => {
     resolve({
       value: 'Result: ' + result,
       done: false,
     });
  });
});
```

如果非同步 Generator 函式丟擲錯誤，會導致 Promise 物件的狀態變為`reject`，然後丟擲的錯誤被`catch`方法捕獲。

```javascript
async function* asyncGenerator() {
  throw new Error('Problem!');
}

asyncGenerator()
.next()
.catch(err => console.log(err)); // Error: Problem!
```

注意，普通的 async 函式返回的是一個 Promise 物件，而非同步 Generator 函式返回的是一個非同步 Iterator 物件。可以這樣理解，async 函式和非同步 Generator 函式，是封裝非同步操作的兩種方法，都用來達到同一種目的。區別在於，前者自帶執行器，後者透過`for await...of`執行，或者自己編寫執行器。下面就是一個非同步 Generator 函式的執行器。

```javascript
async function takeAsync(asyncIterable, count = Infinity) {
  const result = [];
  const iterator = asyncIterable[Symbol.asyncIterator]();
  while (result.length < count) {
    const {value, done} = await iterator.next();
    if (done) break;
    result.push(value);
  }
  return result;
}
```

上面程式碼中，非同步 Generator 函式產生的非同步遍歷器，會透過`while`迴圈自動執行，每當`await iterator.next()`完成，就會進入下一輪迴圈。一旦`done`屬性變為`true`，就會跳出迴圈，非同步遍歷器執行結束。

下面是這個自動執行器的一個使用例項。

```javascript
async function f() {
  async function* gen() {
    yield 'a';
    yield 'b';
    yield 'c';
  }

  return await takeAsync(gen());
}

f().then(function (result) {
  console.log(result); // ['a', 'b', 'c']
})
```

非同步 Generator 函數出現以後，JavaScript 就有了四種函式形式：普通函式、async 函式、Generator 函式和非同步 Generator 函式。請注意區分每種函式的不同之處。基本上，如果是一系列按照順序執行的非同步操作（比如讀取檔案，然後寫入新內容，再存入硬碟），可以使用 async 函式；如果是一系列產生相同資料結構的非同步操作（比如一行一行讀取檔案），可以使用非同步 Generator 函式。

非同步 Generator 函式也可以透過`next`方法的引數，接收外部傳入的資料。

```javascript
const writer = openFile('someFile.txt');
writer.next('hello'); // 立即執行
writer.next('world'); // 立即執行
await writer.return(); // 等待寫入結束
```

上面程式碼中，`openFile`是一個非同步 Generator 函式。`next`方法的引數，向該函式內部的操作傳入資料。每次`next`方法都是同步執行的，最後的`await`命令用於等待整個寫入操作結束。

最後，同步的資料結構，也可以使用非同步 Generator 函式。

```javascript
async function* createAsyncIterable(syncIterable) {
  for (const elem of syncIterable) {
    yield elem;
  }
}
```

上面程式碼中，由於沒有非同步操作，所以也就沒有使用`await`關鍵字。

## yield\* 語句

`yield*`語句也可以跟一個非同步遍歷器。

```javascript
async function* gen1() {
  yield 'a';
  yield 'b';
  return 2;
}

async function* gen2() {
  // result 最終會等於 2
  const result = yield* gen1();
}
```

上面程式碼中，`gen2`函式裡面的`result`變數，最後的值是`2`。

與同步 Generator 函式一樣，`for await...of`迴圈會展開`yield*`。

```javascript
(async function () {
  for await (const x of gen2()) {
    console.log(x);
  }
})();
// a
// b
```

