# 字串的擴充套件

本章介紹 ES6 對字串的改造和增強，下一章介紹字串物件的新增方法。

## 字元的 Unicode 表示法

ES6 加強了對 Unicode 的支援，允許採用`\uxxxx`形式表示一個字元，其中`xxxx`表示字元的 Unicode 碼點。

```javascript
"\u0061"
// "a"
```

但是，這種表示法只限於碼點在`\u0000`~`\uFFFF`之間的字元。超出這個範圍的字元，必須用兩個雙位元組的形式表示。

```javascript
"\uD842\uDFB7"
// "𠮷"

"\u20BB7"
// " 7"
```

上面程式碼表示，如果直接在`\u`後面跟上超過`0xFFFF`的數值（比如`\u20BB7`），JavaScript 會理解成`\u20BB+7`。由於`\u20BB`是一個不可列印字元，所以只會顯示一個空格，後面跟著一個`7`。

ES6 對這一點做出了改進，只要將碼點放入大括號，就能正確解讀該字元。

```javascript
"\u{20BB7}"
// "𠮷"

"\u{41}\u{42}\u{43}"
// "ABC"

let hello = 123;
hell\u{6F} // 123

'\u{1F680}' === '\uD83D\uDE80'
// true
```

上面程式碼中，最後一個例子表明，大括號表示法與四位元組的 UTF-16 編碼是等價的。

有了這種表示法之後，JavaScript 共有 6 種方法可以表示一個字元。

```javascript
'\z' === 'z'  // true
'\172' === 'z' // true
'\x7A' === 'z' // true
'\u007A' === 'z' // true
'\u{7A}' === 'z' // true
```

## 字串的遍歷器介面

ES6 為字串添加了遍歷器介面（詳見《Iterator》一章），使得字串可以被`for...of`迴圈遍歷。

```javascript
for (let codePoint of 'foo') {
  console.log(codePoint)
}
// "f"
// "o"
// "o"
```

除了遍歷字串，這個遍歷器最大的優點是可以識別大於`0xFFFF`的碼點，傳統的`for`迴圈無法識別這樣的碼點。

```javascript
let text = String.fromCodePoint(0x20BB7);

for (let i = 0; i < text.length; i++) {
  console.log(text[i]);
}
// " "
// " "

for (let i of text) {
  console.log(i);
}
// "𠮷"
```

上面程式碼中，字串`text`只有一個字元，但是`for`迴圈會認為它包含兩個字元（都不可列印），而`for...of`迴圈會正確識別出這一個字元。

## 直接輸入 U+2028 和 U+2029

JavaScript 字串允許直接輸入字元，以及輸入字元的轉義形式。舉例來說，“中”的 Unicode 碼點是 U+4e2d，你可以直接在字串裡面輸入這個漢字，也可以輸入它的轉義形式`\u4e2d`，兩者是等價的。

```javascript
'中' === '\u4e2d' // true
```

但是，JavaScript 規定有5個字元，不能在字串裡面直接使用，只能使用轉義形式。

- U+005C：反斜槓（reverse solidus)
- U+000D：回車（carriage return）
- U+2028：行分隔符（line separator）
- U+2029：段分隔符（paragraph separator）
- U+000A：換行符（line feed）

舉例來說，字串裡面不能直接包含反斜槓，一定要轉義寫成`\\`或者`\u005c`。

這個規定本身沒有問題，麻煩在於 JSON 格式允許字串裡面直接使用 U+2028（行分隔符）和 U+2029（段分隔符）。這樣一來，伺服器輸出的 JSON 被`JSON.parse`解析，就有可能直接報錯。

```javascript
const json = '"\u2028"';
JSON.parse(json); // 可能報錯
```

JSON 格式已經凍結（RFC 7159），沒法修改了。為了消除這個報錯，[ES2019](https://github.com/tc39/proposal-json-superset) 允許 JavaScript 字串直接輸入 U+2028（行分隔符）和 U+2029（段分隔符）。

```javascript
const PS = eval("'\u2029'");
```

根據這個提案，上面的程式碼不會報錯。

注意，模板字串現在就允許直接輸入這兩個字元。另外，正則表示式依然不允許直接輸入這兩個字元，這是沒有問題的，因為 JSON 本來就不允許直接包含正則表示式。

## JSON.stringify() 的改造

根據標準，JSON 資料必須是 UTF-8 編碼。但是，現在的`JSON.stringify()`方法有可能返回不符合 UTF-8 標準的字串。

具體來說，UTF-8 標準規定，`0xD800`到`0xDFFF`之間的碼點，不能單獨使用，必須配對使用。比如，`\uD834\uDF06`是兩個碼點，但是必須放在一起配對使用，代表字元`𝌆`。這是為了表示碼點大於`0xFFFF`的字元的一種變通方法。單獨使用`\uD834`和`\uDFO6`這兩個碼點是不合法的，或者顛倒順序也不行，因為`\uDF06\uD834`並沒有對應的字元。

`JSON.stringify()`的問題在於，它可能返回`0xD800`到`0xDFFF`之間的單個碼點。

```javascript
JSON.stringify('\u{D834}') // "\u{D834}"
```

為了確保返回的是合法的 UTF-8 字元，[ES2019](https://github.com/tc39/proposal-well-formed-stringify) 改變了`JSON.stringify()`的行為。如果遇到`0xD800`到`0xDFFF`之間的單個碼點，或者不存在的配對形式，它會返回跳脫字元串，留給應用自己決定下一步的處理。

```javascript
JSON.stringify('\u{D834}') // ""\\uD834""
JSON.stringify('\uDF06\uD834') // ""\\udf06\\ud834""
```

## 模板字串

傳統的 JavaScript 語言，輸出模板通常是這樣寫的（下面使用了 jQuery 的方法）。

```javascript
$('#result').append(
  'There are <b>' + basket.count + '</b> ' +
  'items in your basket, ' +
  '<em>' + basket.onSale +
  '</em> are on sale!'
);
```

上面這種寫法相當繁瑣不方便，ES6 引入了模板字串解決這個問題。

```javascript
$('#result').append(`
  There are <b>${basket.count}</b> items
   in your basket, <em>${basket.onSale}</em>
  are on sale!
`);
```

模板字串（template string）是增強版的字串，用反引號（&#96;）標識。它可以當作普通字串使用，也可以用來定義多行字串，或者在字串中嵌入變數。

```javascript
// 普通字串
`In JavaScript '\n' is a line-feed.`

// 多行字串
`In JavaScript this is
 not legal.`

console.log(`string text line 1
string text line 2`);

// 字串中嵌入變數
let name = "Bob", time = "today";
`Hello ${name}, how are you ${time}?`
```

上面程式碼中的模板字串，都是用反引號表示。如果在模板字串中需要使用反引號，則前面要用反斜槓轉義。

```javascript
let greeting = `\`Yo\` World!`;
```

如果使用模板字串表示多行字串，所有的空格和縮排都會被保留在輸出之中。

```javascript
$('#list').html(`
<ul>
  <li>first</li>
  <li>second</li>
</ul>
`);
```

上面程式碼中，所有模板字串的空格和換行，都是被保留的，比如`<ul>`標籤前面會有一個換行。如果你不想要這個換行，可以使用`trim`方法消除它。

```javascript
$('#list').html(`
<ul>
  <li>first</li>
  <li>second</li>
</ul>
`.trim());
```

模板字串中嵌入變數，需要將變數名寫在`${}`之中。

```javascript
function authorize(user, action) {
  if (!user.hasPrivilege(action)) {
    throw new Error(
      // 傳統寫法為
      // 'User '
      // + user.name
      // + ' is not authorized to do '
      // + action
      // + '.'
      `User ${user.name} is not authorized to do ${action}.`);
  }
}
```

大括號內部可以放入任意的 JavaScript 表示式，可以進行運算，以及引用物件屬性。

```javascript
let x = 1;
let y = 2;

`${x} + ${y} = ${x + y}`
// "1 + 2 = 3"

`${x} + ${y * 2} = ${x + y * 2}`
// "1 + 4 = 5"

let obj = {x: 1, y: 2};
`${obj.x + obj.y}`
// "3"
```

模板字串之中還能呼叫函式。

```javascript
function fn() {
  return "Hello World";
}

`foo ${fn()} bar`
// foo Hello World bar
```

如果大括號中的值不是字串，將按照一般的規則轉為字串。比如，大括號中是一個物件，將預設呼叫物件的`toString`方法。

如果模板字串中的變數沒有宣告，將報錯。

```javascript
// 變數place沒有宣告
let msg = `Hello, ${place}`;
// 報錯
```

由於模板字串的大括號內部，就是執行 JavaScript 程式碼，因此如果大括號內部是一個字串，將會原樣輸出。

```javascript
`Hello ${'World'}`
// "Hello World"
```

模板字串甚至還能巢狀。

```javascript
const tmpl = addrs => `
  <table>
  ${addrs.map(addr => `
    <tr><td>${addr.first}</td></tr>
    <tr><td>${addr.last}</td></tr>
  `).join('')}
  </table>
`;
```

上面程式碼中，模板字串的變數之中，又嵌入了另一個模板字串，使用方法如下。

```javascript
const data = [
    { first: '<Jane>', last: 'Bond' },
    { first: 'Lars', last: '<Croft>' },
];

console.log(tmpl(data));
// <table>
//
//   <tr><td><Jane></td></tr>
//   <tr><td>Bond</td></tr>
//
//   <tr><td>Lars</td></tr>
//   <tr><td><Croft></td></tr>
//
// </table>
```

如果需要引用模板字串本身，在需要時執行，可以寫成函式。

```javascript
let func = (name) => `Hello ${name}!`;
func('Jack') // "Hello Jack!"
```

上面程式碼中，模板字串寫成了一個函式的返回值。執行這個函式，就相當於執行這個模板字串了。

## 例項：模板編譯

下面，我們來看一個透過模板字串，生成正式模板的例項。

```javascript
let template = `
<ul>
  <% for(let i=0; i < data.supplies.length; i++) { %>
    <li><%= data.supplies[i] %></li>
  <% } %>
</ul>
`;
```

上面程式碼在模板字串之中，放置了一個常規模板。該模板使用`<%...%>`放置 JavaScript 程式碼，使用`<%= ... %>`輸出 JavaScript 表示式。

怎麼編譯這個模板字串呢？

一種思路是將其轉換為 JavaScript 表示式字串。

```javascript
echo('<ul>');
for(let i=0; i < data.supplies.length; i++) {
  echo('<li>');
  echo(data.supplies[i]);
  echo('</li>');
};
echo('</ul>');
```

這個轉換使用正則表示式就行了。

```javascript
let evalExpr = /<%=(.+?)%>/g;
let expr = /<%([\s\S]+?)%>/g;

template = template
  .replace(evalExpr, '`); \n  echo( $1 ); \n  echo(`')
  .replace(expr, '`); \n $1 \n  echo(`');

template = 'echo(`' + template + '`);';
```

然後，將`template`封裝在一個函式裡面返回，就可以了。

```javascript
let script =
`(function parse(data){
  let output = "";

  function echo(html){
    output += html;
  }

  ${ template }

  return output;
})`;

return script;
```

將上面的內容拼裝成一個模板編譯函式`compile`。

```javascript
function compile(template){
  const evalExpr = /<%=(.+?)%>/g;
  const expr = /<%([\s\S]+?)%>/g;

  template = template
    .replace(evalExpr, '`); \n  echo( $1 ); \n  echo(`')
    .replace(expr, '`); \n $1 \n  echo(`');

  template = 'echo(`' + template + '`);';

  let script =
  `(function parse(data){
    let output = "";

    function echo(html){
      output += html;
    }

    ${ template }

    return output;
  })`;

  return script;
}
```

`compile`函式的用法如下。

```javascript
let parse = eval(compile(template));
div.innerHTML = parse({ supplies: [ "broom", "mop", "cleaner" ] });
//   <ul>
//     <li>broom</li>
//     <li>mop</li>
//     <li>cleaner</li>
//   </ul>
```

## 標籤模板

模板字串的功能，不僅僅是上面這些。它可以緊跟在一個函式名後面，該函式將被呼叫來處理這個模板字串。這被稱為“標籤模板”功能（tagged template）。

```javascript
alert`hello`
// 等同於
alert(['hello'])
```

標籤模板其實不是模板，而是函式呼叫的一種特殊形式。“標籤”指的就是函式，緊跟在後面的模板字串就是它的引數。

但是，如果模板字元裡面有變數，就不是簡單的呼叫了，而是會將模板字串先處理成多個引數，再呼叫函式。

```javascript
let a = 5;
let b = 10;

tag`Hello ${ a + b } world ${ a * b }`;
// 等同於
tag(['Hello ', ' world ', ''], 15, 50);
```

上面程式碼中，模板字串前面有一個標識名`tag`，它是一個函式。整個表示式的返回值，就是`tag`函式處理模板字串後的返回值。

函式`tag`依次會接收到多個引數。

```javascript
function tag(stringArr, value1, value2){
  // ...
}

// 等同於

function tag(stringArr, ...values){
  // ...
}
```

`tag`函式的第一個引數是一個數組，該陣列的成員是模板字串中那些沒有變數替換的部分，也就是說，變數替換隻發生在陣列的第一個成員與第二個成員之間、第二個成員與第三個成員之間，以此類推。

`tag`函式的其他引數，都是模板字串各個變數被替換後的值。由於本例中，模板字串含有兩個變數，因此`tag`會接受到`value1`和`value2`兩個引數。

`tag`函式所有引數的實際值如下。

- 第一個引數：`['Hello ', ' world ', '']`
- 第二個引數: 15
- 第三個引數：50

也就是說，`tag`函式實際上以下面的形式呼叫。

```javascript
tag(['Hello ', ' world ', ''], 15, 50)
```

我們可以按照需要編寫`tag`函式的程式碼。下面是`tag`函式的一種寫法，以及執行結果。

```javascript
let a = 5;
let b = 10;

function tag(s, v1, v2) {
  console.log(s[0]);
  console.log(s[1]);
  console.log(s[2]);
  console.log(v1);
  console.log(v2);

  return "OK";
}

tag`Hello ${ a + b } world ${ a * b}`;
// "Hello "
// " world "
// ""
// 15
// 50
// "OK"
```

下面是一個更復雜的例子。

```javascript
let total = 30;
let msg = passthru`The total is ${total} (${total*1.05} with tax)`;

function passthru(literals) {
  let result = '';
  let i = 0;

  while (i < literals.length) {
    result += literals[i++];
    if (i < arguments.length) {
      result += arguments[i];
    }
  }

  return result;
}

msg // "The total is 30 (31.5 with tax)"
```

上面這個例子展示了，如何將各個引數按照原來的位置拼合回去。

`passthru`函式採用 rest 引數的寫法如下。

```javascript
function passthru(literals, ...values) {
  let output = "";
  let index;
  for (index = 0; index < values.length; index++) {
    output += literals[index] + values[index];
  }

  output += literals[index]
  return output;
}
```

“標籤模板”的一個重要應用，就是過濾 HTML 字串，防止使用者輸入惡意內容。

```javascript
let message =
  SaferHTML`<p>${sender} has sent you a message.</p>`;

function SaferHTML(templateData) {
  let s = templateData[0];
  for (let i = 1; i < arguments.length; i++) {
    let arg = String(arguments[i]);

    // Escape special characters in the substitution.
    s += arg.replace(/&/g, "&amp;")
            .replace(/</g, "&lt;")
            .replace(/>/g, "&gt;");

    // Don't escape special characters in the template.
    s += templateData[i];
  }
  return s;
}
```

上面程式碼中，`sender`變數往往是使用者提供的，經過`SaferHTML`函式處理，裡面的特殊字元都會被轉義。

```javascript
let sender = '<script>alert("abc")</script>'; // 惡意程式碼
let message = SaferHTML`<p>${sender} has sent you a message.</p>`;

message
// <p>&lt;script&gt;alert("abc")&lt;/script&gt; has sent you a message.</p>
```

標籤模板的另一個應用，就是多語言轉換（國際化處理）。

```javascript
i18n`Welcome to ${siteName}, you are visitor number ${visitorNumber}!`
// "歡迎訪問xxx，您是第xxxx位訪問者！"
```

模板字串本身並不能取代 Mustache 之類的模板庫，因為沒有條件判斷和迴圈處理功能，但是透過標籤函式，你可以自己新增這些功能。

```javascript
// 下面的hashTemplate函式
// 是一個自定義的模板處理函式
let libraryHtml = hashTemplate`
  <ul>
    #for book in ${myBooks}
      <li><i>#{book.title}</i> by #{book.author}</li>
    #end
  </ul>
`;
```

除此之外，你甚至可以使用標籤模板，在 JavaScript 語言之中嵌入其他語言。

```javascript
jsx`
  <div>
    <input
      ref='input'
      onChange='${this.handleChange}'
      defaultValue='${this.state.value}' />
      ${this.state.value}
   </div>
`
```

上面的程式碼透過`jsx`函式，將一個 DOM 字串轉為 React 物件。你可以在 GitHub 找到`jsx`函式的[具體實現](https://gist.github.com/lygaret/a68220defa69174bdec5)。

下面則是一個假想的例子，透過`java`函式，在 JavaScript 程式碼之中執行 Java 程式碼。

```javascript
java`
class HelloWorldApp {
  public static void main(String[] args) {
    System.out.println("Hello World!"); // Display the string.
  }
}
`
HelloWorldApp.main();
```

模板處理函式的第一個引數（模板字串陣列），還有一個`raw`屬性。

```javascript
console.log`123`
// ["123", raw: Array[1]]
```

上面程式碼中，`console.log`接受的引數，實際上是一個數組。該陣列有一個`raw`屬性，儲存的是轉義後的原字串。

請看下面的例子。

```javascript
tag`First line\nSecond line`

function tag(strings) {
  console.log(strings.raw[0]);
  // strings.raw[0] 為 "First line\\nSecond line"
  // 列印輸出 "First line\nSecond line"
}
```

上面程式碼中，`tag`函式的第一個引數`strings`，有一個`raw`屬性，也指向一個數組。該陣列的成員與`strings`陣列完全一致。比如，`strings`陣列是`["First line\nSecond line"]`，那麼`strings.raw`陣列就是`["First line\\nSecond line"]`。兩者唯一的區別，就是字串裡面的斜槓都被轉義了。比如，strings.raw 陣列會將`\n`視為`\\`和`n`兩個字元，而不是換行符。這是為了方便取得轉義之前的原始模板而設計的。

## 模板字串的限制

前面提到標籤模板裡面，可以內嵌其他語言。但是，模板字串預設會將字串轉義，導致無法嵌入其他語言。

舉例來說，標籤模板裡面可以嵌入 LaTEX 語言。

```javascript
function latex(strings) {
  // ...
}

let document = latex`
\newcommand{\fun}{\textbf{Fun!}}  // 正常工作
\newcommand{\unicode}{\textbf{Unicode!}} // 報錯
\newcommand{\xerxes}{\textbf{King!}} // 報錯

Breve over the h goes \u{h}ere // 報錯
`
```

上面程式碼中，變數`document`內嵌的模板字串，對於 LaTEX 語言來說完全是合法的，但是 JavaScript 引擎會報錯。原因就在於字串的轉義。

模板字串會將`\u00FF`和`\u{42}`當作 Unicode 字元進行轉義，所以`\unicode`解析時報錯；而`\x56`會被當作十六進位制字串轉義，所以`\xerxes`會報錯。也就是說，`\u`和`\x`在 LaTEX 裡面有特殊含義，但是 JavaScript 將它們轉義了。

為了解決這個問題，ES2018 [放鬆](https://tc39.github.io/proposal-template-literal-revision/)了對標籤模板裡面的字串轉義的限制。如果遇到不合法的字串轉義，就返回`undefined`，而不是報錯，並且從`raw`屬性上面可以得到原始字串。

```javascript
function tag(strs) {
  strs[0] === undefined
  strs.raw[0] === "\\unicode and \\u{55}";
}
tag`\unicode and \u{55}`
```

上面程式碼中，模板字串原本是應該報錯的，但是由於放鬆了對字串轉義的限制，所以不報錯了，JavaScript 引擎將第一個字元設定為`undefined`，但是`raw`屬性依然可以得到原始字串，因此`tag`函式還是可以對原字串進行處理。

注意，這種對字串轉義的放鬆，只在標籤模板解析字串時生效，不是標籤模板的場合，依然會報錯。

```javascript
let bad = `bad escape sequence: \unicode`; // 報錯
```
