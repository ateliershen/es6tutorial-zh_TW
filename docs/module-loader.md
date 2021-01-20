# Module 的載入實現

上一章介紹了模組的語法，本章介紹如何在瀏覽器和 Node.js 之中載入 ES6 模組，以及實際開發中經常遇到的一些問題（比如迴圈載入）。

## 瀏覽器載入

### 傳統方法

HTML 網頁中，瀏覽器透過`<script>`標籤載入 JavaScript 指令碼。

```html
<!-- 頁面內嵌的指令碼 -->
<script type="application/javascript">
  // module code
</script>

<!-- 外部指令碼 -->
<script type="application/javascript" src="path/to/myModule.js">
</script>
```

上面程式碼中，由於瀏覽器指令碼的預設語言是 JavaScript，因此`type="application/javascript"`可以省略。

預設情況下，瀏覽器是同步載入 JavaScript 指令碼，即渲染引擎遇到`<script>`標籤就會停下來，等到執行完指令碼，再繼續向下渲染。如果是外部指令碼，還必須加入指令碼下載的時間。

如果指令碼體積很大，下載和執行的時間就會很長，因此造成瀏覽器堵塞，使用者會感覺到瀏覽器“卡死”了，沒有任何響應。這顯然是很不好的體驗，所以瀏覽器允許指令碼非同步載入，下面就是兩種非同步載入的語法。

```html
<script src="path/to/myModule.js" defer></script>
<script src="path/to/myModule.js" async></script>
```

上面程式碼中，`<script>`標籤開啟`defer`或`async`屬性，指令碼就會非同步載入。渲染引擎遇到這一行命令，就會開始下載外部指令碼，但不會等它下載和執行，而是直接執行後面的命令。

`defer`與`async`的區別是：`defer`要等到整個頁面在記憶體中正常渲染結束（DOM 結構完全生成，以及其他指令碼執行完成），才會執行；`async`一旦下載完，渲染引擎就會中斷渲染，執行這個指令碼以後，再繼續渲染。一句話，`defer`是“渲染完再執行”，`async`是“下載完就執行”。另外，如果有多個`defer`指令碼，會按照它們在頁面出現的順序載入，而多個`async`指令碼是不能保證載入順序的。

### 載入規則

瀏覽器載入 ES6 模組，也使用`<script>`標籤，但是要加入`type="module"`屬性。

```html
<script type="module" src="./foo.js"></script>
```

上面程式碼在網頁中插入一個模組`foo.js`，由於`type`屬性設為`module`，所以瀏覽器知道這是一個 ES6 模組。

瀏覽器對於帶有`type="module"`的`<script>`，都是非同步載入，不會造成堵塞瀏覽器，即等到整個頁面渲染完，再執行模組指令碼，等同於打開了`<script>`標籤的`defer`屬性。

```html
<script type="module" src="./foo.js"></script>
<!-- 等同於 -->
<script type="module" src="./foo.js" defer></script>
```

如果網頁有多個`<script type="module">`，它們會按照在頁面出現的順序依次執行。

`<script>`標籤的`async`屬性也可以開啟，這時只要載入完成，渲染引擎就會中斷渲染立即執行。執行完成後，再恢復渲染。

```html
<script type="module" src="./foo.js" async></script>
```

一旦使用了`async`屬性，`<script type="module">`就不會按照在頁面出現的順序執行，而是隻要該模組載入完成，就執行該模組。

ES6 模組也允許內嵌在網頁中，語法行為與載入外部指令碼完全一致。

```html
<script type="module">
  import utils from "./utils.js";

  // other code
</script>
```

舉例來說，jQuery 就支援模組載入。

```html
<script type="module">
  import $ from "./jquery/src/jquery.js";
  $('#message').text('Hi from jQuery!');
</script>
```

對於外部的模組指令碼（上例是`foo.js`），有幾點需要注意。

- 程式碼是在模組作用域之中執行，而不是在全域性作用域執行。模組內部的頂層變數，外部不可見。
- 模組指令碼自動採用嚴格模式，不管有沒有宣告`use strict`。
- 模組之中，可以使用`import`命令載入其他模組（`.js`字尾不可省略，需要提供絕對 URL 或相對 URL），也可以使用`export`命令輸出對外介面。
- 模組之中，頂層的`this`關鍵字返回`undefined`，而不是指向`window`。也就是說，在模組頂層使用`this`關鍵字，是無意義的。
- 同一個模組如果載入多次，將只執行一次。

下面是一個示例模組。

```javascript
import utils from 'https://example.com/js/utils.js';

const x = 1;

console.log(x === window.x); //false
console.log(this === undefined); // true
```

利用頂層的`this`等於`undefined`這個語法點，可以偵測當前程式碼是否在 ES6 模組之中。

```javascript
const isNotModuleScript = this !== undefined;
```

## ES6 模組與 CommonJS 模組的差異

討論 Node.js 載入 ES6 模組之前，必須瞭解 ES6 模組與 CommonJS 模組完全不同。

它們有三個重大差異。

- CommonJS 模組輸出的是一個值的複製，ES6 模組輸出的是值的引用。
- CommonJS 模組是執行時載入，ES6 模組是編譯時輸出介面。
- CommonJS 模組的`require()`是同步載入模組，ES6 模組的`import`命令是非同步載入，有一個獨立的模組依賴的解析階段。

第二個差異是因為 CommonJS 載入的是一個物件（即`module.exports`屬性），該物件只有在指令碼執行完才會生成。而 ES6 模組不是物件，它的對外介面只是一種靜態定義，在程式碼靜態解析階段就會生成。

下面重點解釋第一個差異。

CommonJS 模組輸出的是值的複製，也就是說，一旦輸出一個值，模組內部的變化就影響不到這個值。請看下面這個模組檔案`lib.js`的例子。

```javascript
// lib.js
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  counter: counter,
  incCounter: incCounter,
};
```

上面程式碼輸出內部變數`counter`和改寫這個變數的內部方法`incCounter`。然後，在`main.js`裡面載入這個模組。

```javascript
// main.js
var mod = require('./lib');

console.log(mod.counter);  // 3
mod.incCounter();
console.log(mod.counter); // 3
```

上面程式碼說明，`lib.js`模組載入以後，它的內部變化就影響不到輸出的`mod.counter`了。這是因為`mod.counter`是一個原始型別的值，會被快取。除非寫成一個函式，才能得到內部變動後的值。

```javascript
// lib.js
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  get counter() {
    return counter
  },
  incCounter: incCounter,
};
```

上面程式碼中，輸出的`counter`屬性實際上是一個取值器函式。現在再執行`main.js`，就可以正確讀取內部變數`counter`的變動了。

```bash
$ node main.js
3
4
```

ES6 模組的執行機制與 CommonJS 不一樣。JS 引擎對指令碼靜態分析的時候，遇到模組載入命令`import`，就會生成一個只讀引用。等到指令碼真正執行時，再根據這個只讀引用，到被載入的那個模組裡面去取值。換句話說，ES6 的`import`有點像 Unix 系統的“符號連線”，原始值變了，`import`載入的值也會跟著變。因此，ES6 模組是動態引用，並且不會快取值，模組裡面的變數繫結其所在的模組。

還是舉上面的例子。

```javascript
// lib.js
export let counter = 3;
export function incCounter() {
  counter++;
}

// main.js
import { counter, incCounter } from './lib';
console.log(counter); // 3
incCounter();
console.log(counter); // 4
```

上面程式碼說明，ES6 模組輸入的變數`counter`是活的，完全反應其所在模組`lib.js`內部的變化。

再舉一個出現在`export`一節中的例子。

```javascript
// m1.js
export var foo = 'bar';
setTimeout(() => foo = 'baz', 500);

// m2.js
import {foo} from './m1.js';
console.log(foo);
setTimeout(() => console.log(foo), 500);
```

上面程式碼中，`m1.js`的變數`foo`，在剛載入時等於`bar`，過了 500 毫秒，又變為等於`baz`。

讓我們看看，`m2.js`能否正確讀取這個變化。

```bash
$ babel-node m2.js

bar
baz
```

上面程式碼表明，ES6 模組不會快取執行結果，而是動態地去被載入的模組取值，並且變數總是繫結其所在的模組。

由於 ES6 輸入的模組變數，只是一個“符號連線”，所以這個變數是隻讀的，對它進行重新賦值會報錯。

```javascript
// lib.js
export let obj = {};

// main.js
import { obj } from './lib';

obj.prop = 123; // OK
obj = {}; // TypeError
```

上面程式碼中，`main.js`從`lib.js`輸入變數`obj`，可以對`obj`新增屬性，但是重新賦值就會報錯。因為變數`obj`指向的地址是隻讀的，不能重新賦值，這就好比`main.js`創造了一個名為`obj`的`const`變數。

最後，`export`透過介面，輸出的是同一個值。不同的指令碼載入這個介面，得到的都是同樣的例項。

```javascript
// mod.js
function C() {
  this.sum = 0;
  this.add = function () {
    this.sum += 1;
  };
  this.show = function () {
    console.log(this.sum);
  };
}

export let c = new C();
```

上面的指令碼`mod.js`，輸出的是一個`C`的例項。不同的指令碼載入這個模組，得到的都是同一個例項。

```javascript
// x.js
import {c} from './mod';
c.add();

// y.js
import {c} from './mod';
c.show();

// main.js
import './x';
import './y';
```

現在執行`main.js`，輸出的是`1`。

```bash
$ babel-node main.js
1
```

這就證明了`x.js`和`y.js`載入的都是`C`的同一個例項。

## Node.js 的模組載入方法

### 概述

JavaScript 現在有兩種模組。一種是 ES6 模組，簡稱 ESM；另一種是 CommonJS 模組，簡稱 CJS。

CommonJS 模組是 Node.js 專用的，與 ES6 模組不相容。語法上面，兩者最明顯的差異是，CommonJS 模組使用`require()`和`module.exports`，ES6 模組使用`import`和`export`。

它們採用不同的載入方案。從 Node.js v13.2 版本開始，Node.js 已經預設打開了 ES6 模組支援。

Node.js 要求 ES6 模組採用`.mjs`字尾檔名。也就是說，只要指令碼檔案裡面使用`import`或者`export`命令，那麼就必須採用`.mjs`字尾名。Node.js 遇到`.mjs`檔案，就認為它是 ES6 模組，預設啟用嚴格模式，不必在每個模組檔案頂部指定`"use strict"`。

如果不希望將字尾名改成`.mjs`，可以在專案的`package.json`檔案中，指定`type`欄位為`module`。

```javascript
{
   "type": "module"
}
```

一旦設定了以後，該目錄裡面的 JS 指令碼，就被解釋用 ES6 模組。

```bash
# 解釋成 ES6 模組
$ node my-app.js
```

如果這時還要使用 CommonJS 模組，那麼需要將 CommonJS 指令碼的字尾名都改成`.cjs`。如果沒有`type`欄位，或者`type`欄位為`commonjs`，則`.js`指令碼會被解釋成 CommonJS 模組。

總結為一句話：`.mjs`檔案總是以 ES6 模組載入，`.cjs`檔案總是以 CommonJS 模組載入，`.js`檔案的載入取決於`package.json`裡面`type`欄位的設定。

注意，ES6 模組與 CommonJS 模組儘量不要混用。`require`命令不能載入`.mjs`檔案，會報錯，只有`import`命令才可以載入`.mjs`檔案。反過來，`.mjs`檔案裡面也不能使用`require`命令，必須使用`import`。

### package.json 的 main 欄位

`package.json`檔案有兩個欄位可以指定模組的入口檔案：`main`和`exports`。比較簡單的模組，可以只使用`main`欄位，指定模組載入的入口檔案。

```javascript
// ./node_modules/es-module-package/package.json
{
  "type": "module",
  "main": "./src/index.js"
}
```

上面程式碼指定專案的入口指令碼為`./src/index.js`，它的格式為 ES6 模組。如果沒有`type`欄位，`index.js`就會被解釋為 CommonJS 模組。

然後，`import`命令就可以載入這個模組。

```javascript
// ./my-app.mjs

import { something } from 'es-module-package';
// 實際載入的是 ./node_modules/es-module-package/src/index.js
```

上面程式碼中，執行該指令碼以後，Node.js 就會到`./node_modules`目錄下面，尋找`es-module-package`模組，然後根據該模組`package.json`的`main`欄位去執行入口檔案。

這時，如果用 CommonJS 模組的`require()`命令去載入`es-module-package`模組會報錯，因為 CommonJS 模組不能處理`export`命令。

### package.json 的 exports 欄位

`exports`欄位的優先順序高於`main`欄位。它有多種用法。

（1）子目錄別名

`package.json`檔案的`exports`欄位可以指定指令碼或子目錄的別名。

```javascript
// ./node_modules/es-module-package/package.json
{
  "exports": {
    "./submodule": "./src/submodule.js"
  }
}
```

上面的程式碼指定`src/submodule.js`別名為`submodule`，然後就可以從別名載入這個檔案。

```javascript
import submodule from 'es-module-package/submodule';
// 載入 ./node_modules/es-module-package/src/submodule.js
```

下面是子目錄別名的例子。

```javascript
// ./node_modules/es-module-package/package.json
{
  "exports": {
    "./features/": "./src/features/"
  }
}

import feature from 'es-module-package/features/x.js';
// 載入 ./node_modules/es-module-package/src/features/x.js
```

如果沒有指定別名，就不能用“模組+指令碼名”這種形式載入指令碼。

```javascript
// 報錯
import submodule from 'es-module-package/private-module.js';

// 不報錯
import submodule from './node_modules/es-module-package/private-module.js';
```

（2）main 的別名

`exports`欄位的別名如果是`.`，就代表模組的主入口，優先順序高於`main`欄位，並且可以直接簡寫成`exports`欄位的值。

```javascript
{
  "exports": {
    ".": "./main.js"
  }
}

// 等同於
{
  "exports": "./main.js"
}
```

由於`exports`欄位只有支援 ES6 的 Node.js 才認識，所以可以用來相容舊版本的 Node.js。

```javascript
{
  "main": "./main-legacy.cjs",
  "exports": {
    ".": "./main-modern.cjs"
  }
}
```

上面程式碼中，老版本的 Node.js （不支援 ES6 模組）的入口檔案是`main-legacy.cjs`，新版本的 Node.js 的入口檔案是`main-modern.cjs`。

**（3）條件載入**

利用`.`這個別名，可以為 ES6 模組和 CommonJS 指定不同的入口。目前，這個功能需要在 Node.js 執行的時候，開啟`--experimental-conditional-exports`標誌。

```javascript
{
  "type": "module",
  "exports": {
    ".": {
      "require": "./main.cjs",
      "default": "./main.js"
    }
  }
}
```

上面程式碼中，別名`.`的`require`條件指定`require()`命令的入口檔案（即 CommonJS 的入口），`default`條件指定其他情況的入口（即 ES6 的入口）。

上面的寫法可以簡寫如下。

```javascript
{
  "exports": {
    "require": "./main.cjs",
    "default": "./main.js"
  }
}
```

注意，如果同時還有其他別名，就不能採用簡寫，否則或報錯。

```javascript
{
  // 報錯
  "exports": {
    "./feature": "./lib/feature.js",
    "require": "./main.cjs",
    "default": "./main.js"
  }
}
```

### CommonJS 模組載入 ES6 模組

CommonJS 的`require()`命令不能載入 ES6 模組，會報錯，只能使用`import()`這個方法載入。

```javascript
(async () => {
  await import('./my-app.mjs');
})();
```

上面程式碼可以在 CommonJS 模組中執行。

`require()`不支援 ES6 模組的一個原因是，它是同步載入，而 ES6 模組內部可以使用頂層`await`命令，導致無法被同步載入。

### ES6 模組載入 CommonJS 模組

ES6 模組的`import`命令可以載入 CommonJS 模組，但是隻能整體載入，不能只加載單一的輸出項。

```javascript
// 正確
import packageMain from 'commonjs-package';

// 報錯
import { method } from 'commonjs-package';
```

這是因為 ES6 模組需要支援靜態程式碼分析，而 CommonJS 模組的輸出介面是`module.exports`，是一個物件，無法被靜態分析，所以只能整體載入。

載入單一的輸出項，可以寫成下面這樣。

```javascript
import packageMain from 'commonjs-package';
const { method } = packageMain;
```

還有一種變通的載入方法，就是使用 Node.js 內建的`module.createRequire()`方法。

```javascript
// cjs.cjs
module.exports = 'cjs';

// esm.mjs
import { createRequire } from 'module';

const require = createRequire(import.meta.url);

const cjs = require('./cjs.cjs');
cjs === 'cjs'; // true
```

上面程式碼中，ES6 模組透過`module.createRequire()`方法可以載入 CommonJS 模組。但是，這種寫法等於將 ES6 和 CommonJS 混在一起了，所以不建議使用。

### 同時支援兩種格式的模組

一個模組同時要支援 CommonJS 和 ES6 兩種格式，也很容易。

如果原始模組是 ES6 格式，那麼需要給出一個整體輸出介面，比如`export default obj`，使得 CommonJS 可以用`import()`進行載入。

如果原始模組是 CommonJS 格式，那麼可以加一個包裝層。

```javascript
import cjsModule from '../index.js';
export const foo = cjsModule.foo;
```

上面程式碼先整體輸入 CommonJS 模組，然後再根據需要輸出具名介面。

你可以把這個檔案的字尾名改為`.mjs`，或者將它放在一個子目錄，再在這個子目錄裡面放一個單獨的`package.json`檔案，指明`{ type: "module" }`。

另一種做法是在`package.json`檔案的`exports`欄位，指明兩種格式模組各自的載入入口。

```javascript
"exports"：{
  "require": "./index.js"，
  "import": "./esm/wrapper.js"
}
```

上面程式碼指定`require()`和`import`，載入該模組會自動切換到不一樣的入口檔案。

### Node.js 的內建模組

Node.js 的內建模組可以整體載入，也可以載入指定的輸出項。

```javascript
// 整體載入
import EventEmitter from 'events';
const e = new EventEmitter();

// 載入指定的輸出項
import { readFile } from 'fs';
readFile('./foo.txt', (err, source) => {
  if (err) {
    console.error(err);
  } else {
    console.log(source);
  }
});
```

### 載入路徑

ES6 模組的載入路徑必須給出指令碼的完整路徑，不能省略指令碼的字尾名。`import`命令和`package.json`檔案的`main`欄位如果省略指令碼的字尾名，會報錯。

```javascript
// ES6 模組中將報錯
import { something } from './index';
```

為了與瀏覽器的`import`載入規則相同，Node.js 的`.mjs`檔案支援 URL 路徑。

```javascript
import './foo.mjs?query=1'; // 載入 ./foo 傳入引數 ?query=1
```

上面程式碼中，指令碼路徑帶有引數`?query=1`，Node 會按 URL 規則解讀。同一個指令碼只要引數不同，就會被載入多次，並且儲存成不同的快取。由於這個原因，只要檔名中含有`:`、`%`、`#`、`?`等特殊字元，最好對這些字元進行轉義。

目前，Node.js 的`import`命令只支援載入本地模組（`file:`協議）和`data:`協議，不支援載入遠端模組。另外，指令碼路徑只支援相對路徑，不支援絕對路徑（即以`/`或`//`開頭的路徑）。

### 內部變數

ES6 模組應該是通用的，同一個模組不用修改，就可以用在瀏覽器環境和伺服器環境。為了達到這個目標，Node.js 規定 ES6 模組之中不能使用 CommonJS 模組的特有的一些內部變數。

首先，就是`this`關鍵字。ES6 模組之中，頂層的`this`指向`undefined`；CommonJS 模組的頂層`this`指向當前模組，這是兩者的一個重大差異。

其次，以下這些頂層變數在 ES6 模組之中都是不存在的。

- `arguments`
- `require`
- `module`
- `exports`
- `__filename`
- `__dirname`

## 迴圈載入

“迴圈載入”（circular dependency）指的是，`a`指令碼的執行依賴`b`指令碼，而`b`指令碼的執行又依賴`a`指令碼。

```javascript
// a.js
var b = require('b');

// b.js
var a = require('a');
```

通常，“迴圈載入”表示存在強耦合，如果處理不好，還可能導致遞迴載入，使得程式無法執行，因此應該避免出現。

但是實際上，這是很難避免的，尤其是依賴關係複雜的大專案，很容易出現`a`依賴`b`，`b`依賴`c`，`c`又依賴`a`這樣的情況。這意味著，模組載入機制必須考慮“迴圈載入”的情況。

對於 JavaScript 語言來說，目前最常見的兩種模組格式 CommonJS 和 ES6，處理“迴圈載入”的方法是不一樣的，返回的結果也不一樣。

### CommonJS 模組的載入原理

介紹 ES6 如何處理“迴圈載入”之前，先介紹目前最流行的 CommonJS 模組格式的載入原理。

CommonJS 的一個模組，就是一個指令碼檔案。`require`命令第一次載入該指令碼，就會執行整個指令碼，然後在記憶體生成一個物件。

```javascript
{
  id: '...',
  exports: { ... },
  loaded: true,
  ...
}
```

上面程式碼就是 Node 內部載入模組後生成的一個物件。該物件的`id`屬性是模組名，`exports`屬性是模組輸出的各個介面，`loaded`屬性是一個布林值，表示該模組的指令碼是否執行完畢。其他還有很多屬性，這裡都省略了。

以後需要用到這個模組的時候，就會到`exports`屬性上面取值。即使再次執行`require`命令，也不會再次執行該模組，而是到快取之中取值。也就是說，CommonJS 模組無論載入多少次，都只會在第一次載入時執行一次，以後再載入，就返回第一次執行的結果，除非手動清除系統快取。

### CommonJS 模組的迴圈載入

CommonJS 模組的重要特性是載入時執行，即指令碼程式碼在`require`的時候，就會全部執行。一旦出現某個模組被"迴圈載入"，就只輸出已經執行的部分，還未執行的部分不會輸出。

讓我們來看，Node [官方文件](https://nodejs.org/api/modules.html#modules_cycles)裡面的例子。指令碼檔案`a.js`程式碼如下。

```javascript
exports.done = false;
var b = require('./b.js');
console.log('在 a.js 之中，b.done = %j', b.done);
exports.done = true;
console.log('a.js 執行完畢');
```

上面程式碼之中，`a.js`指令碼先輸出一個`done`變數，然後載入另一個指令碼檔案`b.js`。注意，此時`a.js`程式碼就停在這裡，等待`b.js`執行完畢，再往下執行。

再看`b.js`的程式碼。

```javascript
exports.done = false;
var a = require('./a.js');
console.log('在 b.js 之中，a.done = %j', a.done);
exports.done = true;
console.log('b.js 執行完畢');
```

上面程式碼之中，`b.js`執行到第二行，就會去載入`a.js`，這時，就發生了“迴圈載入”。系統會去`a.js`模組對應物件的`exports`屬性取值，可是因為`a.js`還沒有執行完，從`exports`屬性只能取回已經執行的部分，而不是最後的值。

`a.js`已經執行的部分，只有一行。

```javascript
exports.done = false;
```

因此，對於`b.js`來說，它從`a.js`只輸入一個變數`done`，值為`false`。

然後，`b.js`接著往下執行，等到全部執行完畢，再把執行權交還給`a.js`。於是，`a.js`接著往下執行，直到執行完畢。我們寫一個指令碼`main.js`，驗證這個過程。

```javascript
var a = require('./a.js');
var b = require('./b.js');
console.log('在 main.js 之中, a.done=%j, b.done=%j', a.done, b.done);
```

執行`main.js`，執行結果如下。

```bash
$ node main.js

在 b.js 之中，a.done = false
b.js 執行完畢
在 a.js 之中，b.done = true
a.js 執行完畢
在 main.js 之中, a.done=true, b.done=true
```

上面的程式碼證明了兩件事。一是，在`b.js`之中，`a.js`沒有執行完畢，只執行了第一行。二是，`main.js`執行到第二行時，不會再次執行`b.js`，而是輸出快取的`b.js`的執行結果，即它的第四行。

```javascript
exports.done = true;
```

總之，CommonJS 輸入的是被輸出值的複製，不是引用。

另外，由於 CommonJS 模組遇到迴圈載入時，返回的是當前已經執行的部分的值，而不是程式碼全部執行後的值，兩者可能會有差異。所以，輸入變數的時候，必須非常小心。

```javascript
var a = require('a'); // 安全的寫法
var foo = require('a').foo; // 危險的寫法

exports.good = function (arg) {
  return a.foo('good', arg); // 使用的是 a.foo 的最新值
};

exports.bad = function (arg) {
  return foo('bad', arg); // 使用的是一個部分載入時的值
};
```

上面程式碼中，如果發生迴圈載入，`require('a').foo`的值很可能後面會被改寫，改用`require('a')`會更保險一點。

### ES6 模組的迴圈載入

ES6 處理“迴圈載入”與 CommonJS 有本質的不同。ES6 模組是動態引用，如果使用`import`從一個模組載入變數（即`import foo from 'foo'`），那些變數不會被快取，而是成為一個指向被載入模組的引用，需要開發者自己保證，真正取值的時候能夠取到值。

請看下面這個例子。

```javascript
// a.mjs
import {bar} from './b';
console.log('a.mjs');
console.log(bar);
export let foo = 'foo';

// b.mjs
import {foo} from './a';
console.log('b.mjs');
console.log(foo);
export let bar = 'bar';
```

上面程式碼中，`a.mjs`載入`b.mjs`，`b.mjs`又載入`a.mjs`，構成迴圈載入。執行`a.mjs`，結果如下。

```bash
$ node --experimental-modules a.mjs
b.mjs
ReferenceError: foo is not defined
```

上面程式碼中，執行`a.mjs`以後會報錯，`foo`變數未定義，這是為什麼？

讓我們一行行來看，ES6 迴圈載入是怎麼處理的。首先，執行`a.mjs`以後，引擎發現它載入了`b.mjs`，因此會優先執行`b.mjs`，然後再執行`a.mjs`。接著，執行`b.mjs`的時候，已知它從`a.mjs`輸入了`foo`介面，這時不會去執行`a.mjs`，而是認為這個介面已經存在了，繼續往下執行。執行到第三行`console.log(foo)`的時候，才發現這個介面根本沒定義，因此報錯。

解決這個問題的方法，就是讓`b.mjs`執行的時候，`foo`已經有定義了。這可以透過將`foo`寫成函式來解決。

```javascript
// a.mjs
import {bar} from './b';
console.log('a.mjs');
console.log(bar());
function foo() { return 'foo' }
export {foo};

// b.mjs
import {foo} from './a';
console.log('b.mjs');
console.log(foo());
function bar() { return 'bar' }
export {bar};
```

這時再執行`a.mjs`就可以得到預期結果。

```bash
$ node --experimental-modules a.mjs
b.mjs
foo
a.mjs
bar
```

這是因為函式具有提升作用，在執行`import {bar} from './b'`時，函式`foo`就已經有定義了，所以`b.mjs`載入的時候不會報錯。這也意味著，如果把函式`foo`改寫成函式表示式，也會報錯。

```javascript
// a.mjs
import {bar} from './b';
console.log('a.mjs');
console.log(bar());
const foo = () => 'foo';
export {foo};
```

上面程式碼的第四行，改成了函式表示式，就不具有提升作用，執行就會報錯。

我們再來看 ES6 模組載入器[SystemJS](https://github.com/ModuleLoader/es6-module-loader/blob/master/docs/circular-references-bindings.md)給出的一個例子。

```javascript
// even.js
import { odd } from './odd'
export var counter = 0;
export function even(n) {
  counter++;
  return n === 0 || odd(n - 1);
}

// odd.js
import { even } from './even';
export function odd(n) {
  return n !== 0 && even(n - 1);
}
```

上面程式碼中，`even.js`裡面的函式`even`有一個引數`n`，只要不等於 0，就會減去 1，傳入載入的`odd()`。`odd.js`也會做類似操作。

執行上面這段程式碼，結果如下。

```javascript
$ babel-node
> import * as m from './even.js';
> m.even(10);
true
> m.counter
6
> m.even(20)
true
> m.counter
17
```

上面程式碼中，引數`n`從 10 變為 0 的過程中，`even()`一共會執行 6 次，所以變數`counter`等於 6。第二次呼叫`even()`時，引數`n`從 20 變為 0，`even()`一共會執行 11 次，加上前面的 6 次，所以變數`counter`等於 17。

這個例子要是改寫成 CommonJS，就根本無法執行，會報錯。

```javascript
// even.js
var odd = require('./odd');
var counter = 0;
exports.counter = counter;
exports.even = function (n) {
  counter++;
  return n == 0 || odd(n - 1);
}

// odd.js
var even = require('./even').even;
module.exports = function (n) {
  return n != 0 && even(n - 1);
}
```

上面程式碼中，`even.js`載入`odd.js`，而`odd.js`又去載入`even.js`，形成“迴圈載入”。這時，執行引擎就會輸出`even.js`已經執行的部分（不存在任何結果），所以在`odd.js`之中，變數`even`等於`undefined`，等到後面呼叫`even(n - 1)`就會報錯。

```bash
$ node
> var m = require('./even');
> m.even(10)
TypeError: even is not a function
```

