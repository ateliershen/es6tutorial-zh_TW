# ECMAScript 6 簡介

ECMAScript 6.0（以下簡稱 ES6）是 JavaScript 語言的下一代標準，已經在 2015 年 6 月正式釋出了。它的目標，是使得 JavaScript 語言可以用來編寫複雜的大型應用程式，成為企業級開發語言。

## ECMAScript 和 JavaScript 的關係

一個常見的問題是，ECMAScript 和 JavaScript 到底是什麼關係？

要講清楚這個問題，需要回顧歷史。1996 年 11 月，JavaScript 的創造者 Netscape 公司，決定將 JavaScript 提交給標準化組織 ECMA，希望這種語言能夠成為國際標準。次年，ECMA 釋出 262 號標準檔案（ECMA-262）的第一版，規定了瀏覽器指令碼語言的標準，並將這種語言稱為 ECMAScript，這個版本就是 1.0 版。

該標準從一開始就是針對 JavaScript 語言制定的，但是之所以不叫 JavaScript，有兩個原因。一是商標，Java 是 Sun 公司的商標，根據授權協議，只有 Netscape 公司可以合法地使用 JavaScript 這個名字，且 JavaScript 本身也已經被 Netscape 公司註冊為商標。二是想體現這門語言的制定者是 ECMA，不是 Netscape，這樣有利於保證這門語言的開放性和中立性。

因此，ECMAScript 和 JavaScript 的關係是，前者是後者的規格，後者是前者的一種實現（另外的 ECMAScript 方言還有 JScript 和 ActionScript）。日常場合，這兩個詞是可以互換的。

## ES6 與 ECMAScript 2015 的關係

ECMAScript 2015（簡稱 ES2015）這個詞，也是經常可以看到的。它與 ES6 是什麼關係呢？

2011 年，ECMAScript 5.1 版釋出後，就開始制定 6.0 版了。因此，ES6 這個詞的原意，就是指 JavaScript 語言的下一個版本。

但是，因為這個版本引入的語法功能太多，而且制定過程當中，還有很多組織和個人不斷提交新功能。事情很快就變得清楚了，不可能在一個版本里麵包括所有將要引入的功能。常規的做法是先發布 6.0 版，過一段時間再發 6.1 版，然後是 6.2 版、6.3 版等等。

但是，標準的制定者不想這樣做。他們想讓標準的升級成為常規流程：任何人在任何時候，都可以向標準委員會提交新語法的提案，然後標準委員會每個月開一次會，評估這些提案是否可以接受，需要哪些改進。如果經過多次會議以後，一個提案足夠成熟了，就可以正式進入標準了。這就是說，標準的版本升級成為了一個不斷滾動的流程，每個月都會有變動。

標準委員會最終決定，標準在每年的 6 月份正式釋出一次，作為當年的正式版本。接下來的時間，就在這個版本的基礎上做改動，直到下一年的 6 月份，草案就自然變成了新一年的版本。這樣一來，就不需要以前的版本號了，只要用年份標記就可以了。

ES6 的第一個版本，就這樣在 2015 年 6 月釋出了，正式名稱就是《ECMAScript 2015 標準》（簡稱 ES2015）。2016 年 6 月，小幅修訂的《ECMAScript 2016 標準》（簡稱 ES2016）如期釋出，這個版本可以看作是 ES6.1 版，因為兩者的差異非常小（只新增了陣列例項的`includes`方法和指數運算子），基本上是同一個標準。根據計劃，2017 年 6 月釋出 ES2017 標準。

因此，ES6 既是一個歷史名詞，也是一個泛指，含義是 5.1 版以後的 JavaScript 的下一代標準，涵蓋了 ES2015、ES2016、ES2017 等等，而 ES2015 則是正式名稱，特指該年釋出的正式版本的語言標準。本書中提到 ES6 的地方，一般是指 ES2015 標準，但有時也是泛指“下一代 JavaScript 語言”。

## 語法提案的批准流程

任何人都可以向標準委員會（又稱 TC39 委員會）提案，要求修改語言標準。

一種新的語法從提案到變成正式標準，需要經歷五個階段。每個階段的變動都需要由 TC39 委員會批准。

- Stage 0 - Strawman（展示階段）
- Stage 1 - Proposal（徵求意見階段）
- Stage 2 - Draft（草案階段）
- Stage 3 - Candidate（候選人階段）
- Stage 4 - Finished（定案階段）

一個提案只要能進入 Stage 2，就差不多肯定會包括在以後的正式標準裡面。ECMAScript 當前的所有提案，可以在 TC39 的官方網站[GitHub.com/tc39/ecma262](https://github.com/tc39/ecma262)檢視。

本書的寫作目標之一，是跟蹤 ECMAScript 語言的最新進展，介紹 5.1 版本以後所有的新語法。對於那些明確或很有希望，將要列入標準的新語法，都將予以介紹。

## ECMAScript 的歷史

ES6 從開始制定到最後釋出，整整用了 15 年。

前面提到，ECMAScript 1.0 是 1997 年釋出的，接下來的兩年，連續釋出了 ECMAScript 2.0（1998 年 6 月）和 ECMAScript 3.0（1999 年 12 月）。3.0 版是一個巨大的成功，在業界得到廣泛支援，成為通行標準，奠定了 JavaScript 語言的基本語法，以後的版本完全繼承。直到今天，初學者一開始學習 JavaScript，其實就是在學 3.0 版的語法。

2000 年，ECMAScript 4.0 開始醞釀。這個版本最後沒有透過，但是它的大部分內容被 ES6 繼承了。因此，ES6 制定的起點其實是 2000 年。

為什麼 ES4 沒有透過呢？因為這個版本太激進了，對 ES3 做了徹底升級，導致標準委員會的一些成員不願意接受。ECMA 的第 39 號技術專家委員會（Technical Committee 39，簡稱 TC39）負責制訂 ECMAScript 標準，成員包括 Microsoft、Mozilla、Google 等大公司。

2007 年 10 月，ECMAScript 4.0 版草案發布，本來預計次年 8 月釋出正式版本。但是，各方對於是否透過這個標準，發生了嚴重分歧。以 Yahoo、Microsoft、Google 為首的大公司，反對 JavaScript 的大幅升級，主張小幅改動；以 JavaScript 創造者 Brendan Eich 為首的 Mozilla 公司，則堅持當前的草案。

2008 年 7 月，由於對於下一個版本應該包括哪些功能，各方分歧太大，爭論過於激烈，ECMA 開會決定，中止 ECMAScript 4.0 的開發，將其中涉及現有功能改善的一小部分，釋出為 ECMAScript 3.1，而將其他激進的設想擴大範圍，放入以後的版本，由於會議的氣氛，該版本的專案代號起名為 Harmony（和諧）。會後不久，ECMAScript 3.1 就改名為 ECMAScript 5。

2009 年 12 月，ECMAScript 5.0 版正式釋出。Harmony 專案則一分為二，一些較為可行的設想定名為 JavaScript.next 繼續開發，後來演變成 ECMAScript 6；一些不是很成熟的設想，則被視為 JavaScript.next.next，在更遠的將來再考慮推出。TC39 委員會的總體考慮是，ES5 與 ES3 基本保持相容，較大的語法修正和新功能加入，將由 JavaScript.next 完成。當時，JavaScript.next 指的是 ES6，第六版釋出以後，就指 ES7。TC39 的判斷是，ES5 會在 2013 年的年中成為 JavaScript 開發的主流標準，並在此後五年中一直保持這個位置。

2011 年 6 月，ECMAScript 5.1 版釋出，並且成為 ISO 國際標準（ISO/IEC 16262:2011）。

2013 年 3 月，ECMAScript 6 草案凍結，不再新增新功能。新的功能設想將被放到 ECMAScript 7。

2013 年 12 月，ECMAScript 6 草案發布。然後是 12 個月的討論期，聽取各方反饋。

2015 年 6 月，ECMAScript 6 正式透過，成為國際標準。從 2000 年算起，這時已經過去了 15 年。

目前，各大瀏覽器對 ES6 的支援可以檢視[kangax.github.io/compat-table/es6/](https://kangax.github.io/compat-table/es6/)。

Node.js 是 JavaScript 的伺服器執行環境（runtime）。它對 ES6 的支援度更高。除了那些預設開啟的功能，還有一些語法功能已經實現了，但是預設沒有開啟。使用下面的命令，可以檢視 Node.js 預設沒有開啟的 ES6 實驗性語法。

```bash
// Linux & Mac
$ node --v8-options | grep harmony

// Windows
$ node --v8-options | findstr harmony
```

## Babel 轉碼器

[Babel](https://babeljs.io/) 是一個廣泛使用的 ES6 轉碼器，可以將 ES6 程式碼轉為 ES5 程式碼，從而在老版本的瀏覽器執行。這意味著，你可以用 ES6 的方式編寫程式，又不用擔心現有環境是否支援。下面是一個例子。

```javascript
// 轉碼前
input.map(item => item + 1);

// 轉碼後
input.map(function (item) {
  return item + 1;
});
```

上面的原始程式碼用了箭頭函式，Babel 將其轉為普通函式，就能在不支援箭頭函式的 JavaScript 環境執行了。

下面的命令在專案目錄中，安裝 Babel。

```bash
$ npm install --save-dev @babel/core
```

### 配置檔案`.babelrc`

Babel 的配置檔案是`.babelrc`，存放在專案的根目錄下。使用 Babel 的第一步，就是配置這個檔案。

該檔案用來設定轉碼規則和外掛，基本格式如下。

```javascript
{
  "presets": [],
  "plugins": []
}
```

`presets`欄位設定轉碼規則，官方提供以下的規則集，你可以根據需要安裝。

```bash
# 最新轉碼規則
$ npm install --save-dev @babel/preset-env

# react 轉碼規則
$ npm install --save-dev @babel/preset-react
```

然後，將這些規則加入`.babelrc`。

```javascript
  {
    "presets": [
      "@babel/env",
      "@babel/preset-react"
    ],
    "plugins": []
  }
```

注意，以下所有 Babel 工具和模組的使用，都必須先寫好`.babelrc`。

### 命令列轉碼

Babel 提供命令列工具`@babel/cli`，用於命令列轉碼。

它的安裝命令如下。

```bash
$ npm install --save-dev @babel/cli
```

基本用法如下。

```bash
# 轉碼結果輸出到標準輸出
$ npx babel example.js

# 轉碼結果寫入一個檔案
# --out-file 或 -o 引數指定輸出檔案
$ npx babel example.js --out-file compiled.js
# 或者
$ npx babel example.js -o compiled.js

# 整個目錄轉碼
# --out-dir 或 -d 引數指定輸出目錄
$ npx babel src --out-dir lib
# 或者
$ npx babel src -d lib

# -s 引數生成source map檔案
$ npx babel src -d lib -s
```

### babel-node

`@babel/node`模組的`babel-node`命令，提供一個支援 ES6 的 REPL 環境。它支援 Node 的 REPL 環境的所有功能，而且可以直接執行 ES6 程式碼。

首先，安裝這個模組。

```bash
$ npm install --save-dev @babel/node
```

然後，執行`babel-node`就進入 REPL 環境。

```bash
$ npx babel-node
> (x => x * 2)(1)
2
```

`babel-node`命令可以直接執行 ES6 指令碼。將上面的程式碼放入指令碼檔案`es6.js`，然後直接執行。

```bash
# es6.js 的程式碼
# console.log((x => x * 2)(1));
$ npx babel-node es6.js
2
```

### @babel/register 模組

`@babel/register`模組改寫`require`命令，為它加上一個鉤子。此後，每當使用`require`載入`.js`、`.jsx`、`.es`和`.es6`字尾名的檔案，就會先用 Babel 進行轉碼。

```bash
$ npm install --save-dev @babel/register
```

使用時，必須首先載入`@babel/register`。

```bash
// index.js
require('@babel/register');
require('./es6.js');
```

然後，就不需要手動對`index.js`轉碼了。

```bash
$ node index.js
2
```

需要注意的是，`@babel/register`只會對`require`命令載入的檔案轉碼，而不會對當前檔案轉碼。另外，由於它是實時轉碼，所以只適合在開發環境使用。

### polyfill

Babel 預設只轉換新的 JavaScript 句法（syntax），而不轉換新的 API，比如`Iterator`、`Generator`、`Set`、`Map`、`Proxy`、`Reflect`、`Symbol`、`Promise`等全域性物件，以及一些定義在全域性物件上的方法（比如`Object.assign`）都不會轉碼。

舉例來說，ES6 在`Array`物件上新增了`Array.from`方法。Babel 就不會轉碼這個方法。如果想讓這個方法執行，可以使用`core-js`和`regenerator-runtime`(後者提供generator函式的轉碼)，為當前環境提供一個墊片。

安裝命令如下。

```bash
$ npm install --save-dev core-js regenerator-runtime
```

然後，在指令碼頭部，加入如下兩行程式碼。

```javascript
import 'core-js';
import 'regenerator-runtime/runtime';
// 或者
require('core-js');
require('regenerator-runtime/runtime);
```

Babel 預設不轉碼的 API 非常多，詳細清單可以檢視`babel-plugin-transform-runtime`模組的[definitions.js](https://github.com/babel/babel/blob/master/packages/babel-plugin-transform-runtime/src/runtime-corejs3-definitions.js)檔案。

### 瀏覽器環境

Babel 也可以用於瀏覽器環境，使用[@babel/standalone](https://babeljs.io/docs/en/next/babel-standalone.html)模組提供的瀏覽器版本，將其插入網頁。

```html
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
<script type="text/babel">
// Your ES6 code
</script>
```

注意，網頁實時將 ES6 程式碼轉為 ES5，對效能會有影響。生產環境需要載入已經轉碼完成的指令碼。

Babel 提供一個[REPL 線上編譯器](https://babeljs.io/repl/)，可以線上將 ES6 程式碼轉為 ES5 程式碼。轉換後的程式碼，可以直接作為 ES5 程式碼插入網頁執行。

