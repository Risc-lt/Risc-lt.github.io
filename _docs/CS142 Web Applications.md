---
title: "CS142 Web Applications"
excerpt: "为了兼顾课业以及同期开始的Computer Organization，笔者收掉这门课的跨度大概是两个月的时间。最开始的目的只是熟悉基本的框架和操作，但其中的project一脉相承，算是比较不错的练习"
collection: docs
---

# 前言（Intro)

​		为了兼顾课业以及同期开始的Computer Organization，笔者收掉这门课的跨度大概是两个月的时间。最开始的目的只是熟悉基本的框架和操作，但其中的project一脉相承，算是比较不错的练习（唯一的缺点就是Slides过于简略也没有配套视频，以及课程相对冷门缺少一些讨论交流的空间）

![](https://s2.loli.net/2024/01/01/LTuHklJPIa4DnbC.png)

​		打算开始学习这门课程之前，笔者也横向对比过[MIT Web Development Crash Course](https://weblab.mit.edu/schedule/) ，其优势是除Slides外还有配套的视频讲解，不至于像142中需要自己查找大量的资料来辅助理解，当然最重要的一点是，142的框架相较于前者比较单一的前端开发和页面设计更加全面：

```
$ CS142
Content
├── Foundation
│   ├── HTML
│   ├── CSS
│   ├── URLs
│   └── JavaScript
│       ├── DOM
│       └── Events Handling
├── Web Browser
│   ├── ReactJS
│       ├── Router
│       ├── Responsive Layout
│       └── Material-UI
│   ├── HTTP
│   ├── Promises 
├── Web Server
│   ├── Node.js
│       └── ExpressJS
├── Storage System
│   ├── SQL
│   ├── MongoDB
├── StateManagement
│   ├── Sessions
│       └── Cookies
│   ├── Input Validation
│   ├── Listener / Emitter
├── Web App Security
│   ├── Attacks
│   	├── Network
│   	├── Session
│   	├── Code Injection
│   	├── Phishing
│   	└── DOS
├── Outlook
│   ├── Large scale applications
│   ├── Data Centers
│   └── Future Web App Tech
└── End
```

​		Anyway，如果想学习具体的前端开发乃至于全栈开发，笔者并不推荐单看这门课的lecture，因为0基础实操起project的时候确实会有一个很痛苦的起步过程。而笔者写下这个通关记录，既是分享经历，也是想从头梳理一下这个项目，看看自己究竟掌握到了何种程度

# 课程笔记(notes)

## Lecture

##### HTML

​		HTML（Hypertext Markup Language）是一种用于创建和设计网页的标记语言。它使用标签（tag）来描述网页的结构和内容。下面是HTML的基本语法和一些示例标签：

1. **HTML文档结构：**
   一个基本的HTML文档包含以下结构：

   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <title>文档标题</title>
   </head>
   <body>
       <!-- 文档内容 -->
   </body>
   </html>
   ```

   - `<!DOCTYPE html>`：声明HTML版本(此处是**HTML5**)，其他的声明类型还有如```<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">```(**HTML 4.01 Strict:**)
   - `<html>`：HTML文档的根元素。
   - `<head>`：包含文档的元信息，如标题、字符集等。
   - `<title>`：定义文档的标题，显示在浏览器的标题栏中。
   - `<body>`：包含文档的实际内容。

2. **HTML标签：**
   - 标签通常是成对出现的，有开始标签和结束标签。
   - 例如，`<p>` 是一个段落的开始标签，`</p>` 是相应的结束标签。

3. **常见标签示例：**
   - `<h1>到<h6>`：定义标题，`<h1>`是最高级别，`<h6>`是最低级别。
   - `<p>`：定义段落。
   - `<a>`：定义超链接。
   - `<img>`：插入图像。
   - `<ul>`：定义无序列表。
   - `<ol>`：定义有序列表。
   - `<li>`：定义列表项。
   - `<table>`：定义表格。
   - `<tr>`：定义表格行。
   - `<td>`：定义表格数据单元格。
   - `<div>`：定义文档中的一个区块。
   - `<a> `：用于创建超链接。
   
4. **属性：**
   - 标签可以包含属性，属性提供有关标签的额外信息。
   - 例如，`<a href="https://www.example.com">链接</a>` 中的 `href` 是一个属性，指定需要跳转到的链接的目标URL。
   - `<img src="face.jpg">`中的`<img>` 标签用于在网页中插入图像，而``src` 属性指定图像文件的路径或URL。
   - `<input type="text" value="94301" name="zip">`中的`<input>` 标签用于创建表单元素，`type="text"` 指定了输入框的类型为文本框，`value="94301"` 设置了文本框的默认值为 "94301"，``name="zip"` 定义了该输入框的名称，用于在表单提交时标识输入的数据。
   - `<div class="header">`中``<div>` 标签用于定义HTML文档中的一个区块，通常用于组织和布局内容，属性`class="header"` 定义了该 `<div>` 元素的类名为 "header"。类名通常用于通过CSS样式表为元素应用样式。
   
5. **注释：**
   
   - 在HTML中，可以使用`<!-- 这是注释 -->`来添加注释，注释不会在浏览器中显示。

​		类似的，还有**XHTML**和**XML**两种类似的语言，前者在在自定义的标签闭合和大小写敏感等问题上有所不同，后者则需要在文件头部用`<?xml version="1.0" encoding="utf-8"?>`的标记独立声明。

​		For more detalis, please take a deeper look into [HTML](https://developer.mozilla.org/zh-CN/docs/Web/HTML).

##### CSS

CSS（层叠样式表）是一种用于描述文档如何呈现的样式语言。它定义了文档的布局、颜色、字体等方面的样式。以下是CSS的基本语法和一些示例：

1. **CSS规则：**
   - CSS规则由选择器和声明块组成。
   - 选择器指定了样式应用的HTML元素，而声明块包含一个或多个属性-值对，描述了元素的样式。

   ```css
   selector {
       property: value;
       /* 可以有多个属性-值对 */
   }
   ```

2. **选择器：**
   - 选择器用于选择要应用样式的HTML元素。
   - 例如，`h1` 是选择所有 `<h1>` 标签的选择器。

   ```css
   h1 {
       color: blue;
       font-size: 24px;
   }
   ```

3. **属性-值对：**
   - 属性-值对定义了要应用于所选元素的样式。
   - 例如，`color: blue;` 设置所选元素的文本颜色为蓝色。

   ```css
   p {
       font-family: "Arial", sans-serif;
       margin-bottom: 20px;
   }
   ```

4. **注释：**
   - CSS中的注释以 `/*` 开始，以 `*/` 结束。

   ```css
   /* 这是一个注释 */
   p {
       /* 这是另一个注释 */
       color: green;
   }
   ```

5. **常见的CSS属性及其值的示例：**

   1. **颜色相关属性：**
      
      ```css
      /* 设置文本颜色为红色 */
      color: red;
      
      /* 设置背景颜色为浅灰色 */
      background-color: #f0f0f0;
      
      /* 设置边框颜色为黑色 */
      border-color: black;
      ```
      
   2. **字体属性：**
      ```css
      /* 设置字体类型为Arial或sans-serif（如果Arial不可用） */
      font-family: "Arial", sans-serif;
      
      /* 设置字体大小为16像素 */
      font-size: 16px;
      
      /* 设置字体粗细为粗体 */
      font-weight: bold;
      ```

   3. **布局和边距属性：**
      ```css
      /* 设置元素的上外边距为10像素 */
      margin-top: 10px;
      
      /* 设置元素的内边距为15像素 */
      padding: 15px;
      
      /* 设置元素的宽度为50% */
      width: 50%;
      
      /* 设置元素的高度为100像素 */
      height: 100px;
      ```

   4. **定位和浮动属性：**
      ```css
      /* 将元素相对于其正常位置移动，不影响其他元素的布局 */
      position: absolute;
      
      /* 元素浮动到左侧 */
      float: left;
      
      /* 清除浮动效果，确保在浮动元素下方有足够的空间 */
      clear: both;
      ```

   5. **文本属性：**
      ```css
      /* 设置文本水平居中 */
      text-align: center;
      
      /* 设置文本装饰为下划线 */
      text-decoration: underline;
      
      /* 设置行高为1.5倍字体大小 */
      line-height: 1.5;
      ```

6. **引入外部样式表：**

   - 可以使用 `<link>` 元素将外部CSS文件引入HTML文档中。

   ```html
   <head>
       <link rel="stylesheet" type="text/css" href="styles.css">
   </head>
   ```

   其中，`styles.css` 是包含CSS规则的外部样式表文件。

   CSS的规范和属性列表相当庞大，可以在 [CSS](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/CSS_basics)等网站上找到详细的文档。

##### URLs

​		URL（Uniform Resource Locator）是用于定位资源的字符串。URL通常用于指定网络上的资源，如网页、图像、文件等。以下是URL的基本语法和一些示例：

1. **URL的基本结构：**
   
   ```
   scheme://host:port/path?query#fragment
   ```
   
   - `scheme`: 指定协议（例如，`http`、`https`、`ftp`）。
   - `host`: 指定主机名或IP地址。
   - `port`: 指定端口号（可选，默认根据协议使用默认端口）。
   - `path`: 指定资源在服务器上的路径。
   - `query`: 包含参数的查询字符串（可选）。
   - `fragment`: 指定文档中的特定位置（可选）。
   
2. **示例URL：**
   ```plaintext
   https://www.example.com:8080/path/to/resource?param1=value1&param2=value2#section
   ```

   - `scheme`: `https`
   - `host`: `www.example.com`
   - `port`: `8080`
   - `path`: `/path/to/resource`
   - `query`: `param1=value1&param2=value2`
   - `fragment`: `section`

3. **URL编码：**
   - URL中的某些字符（例如空格、特殊字符）需要进行编码，使用 `%` 后跟两个十六进制数字表示。

   ```plaintext
   https://www.example.com/search?q=hello%20world
   ```

   这里 `%20` 表示空格。

![](CS142 Web Applications/VQKbBI6eWdX7gyD.png)

​		URL的具体语法和规范可以在 [RFC 3986](https://datatracker.ietf.org/doc/html/rfc3986) 中找到。URL的语法可以根据具体的应用和场景有所变化，例如，有些URL可能没有端口号、查询字符串或片段。

##### JavaScript

​		JavaScript（JS）是一种用于为网页添加交互性的脚本语言。以下是JavaScript的基本语法和一些示例：

1. **变量声明：**

   - 用 `var`、`let` 或 `const` 关键字声明变量。

   ```javascript
   var x = 5;  // 用 var 声明变量
   let y = 10; // 用 let 声明变量（具有块级作用域）
   const PI = 3.14; // 用 const 声明常量
   ```

2. **数据类型：**
   - JavaScript有七种基本数据类型，分为两类：原始数据类型和对象数据类型。

   **原始数据类型（Primitive Data Types）：**

   1. **字符串（String）：**
      
      - 表示文本数据。
      ```javascript
      var name = "John";
      ```
      
   2. **数字（Number）：**
      
      - 表示数值。
      ```javascript
      var age = 25;
      ```
      
   3. **布尔值（Boolean）：**
      - 表示逻辑值，只有两个值：`true` 或 `false`。
      ```javascript
      var isAdult = true;
      ```

   4. **空值（Null）：**
      - 表示空值或无值。
      ```javascript
      var data = null;
      ```

   5. **未定义（Undefined）：**
      - 表示变量已声明但未赋值的状态。
      ```javascript
      var status;
      ```

   6. **符号（Symbol）：**
      - ES6引入的一种新的数据类型，用于创建唯一的标识符。
      ```javascript
      var id = Symbol("uniqueId");
      ```

   **对象数据类型（Object Data Type）：**

   7. **对象（Object）：**

      - 用于组织和存储数据的复合数据类型。

      ```javascript
      var person = { name: "John", age: 30 };
      ```

   8. 数组（Array）：

      - 用于按顺序存储一组值的列表。

      ```javascript
      var colors = ["red", "green", "blue"];
      ```

   9. 函数（Function）：

      - 函数是可执行的代码块，可以通过名称调用。

      ```javascript
      function add(a, b) {
          return a + b;
      }
      ```

      这些数据类型可以进一步分为**可变**（原始数据类型除外）和**不可变**。对象数据类型是可变的，因为它们的值可以修改，而原始数据类型是不可变的，一旦创建就不能被更改。。

3. **条件语句：**

   - 使用 `if`、`else if`、`else` 进行条件判断。

   ```javascript
   var age = 18;
   if (age < 18) {
       console.log("未成年");
   } else {
       console.log("成年人");
   }
   ```

4. **循环语句：**
   - 使用 `for` 或 `while` 进行循环。

   ```javascript
   for (var i = 0; i < 5; i++) {
       console.log(i);
   }
   
   var counter = 0;
   while (counter < 5) {
       console.log(counter);
       counter++;
   }
   ```

5. **正则表达式**：

   1. `test`方法:

      (1)**检查子串是否存在：**

      ```javascript
      /HALT/.test(str);
      ```
      - 如果字符串 `str` 中包含子串 "HALT"，则返回 `true`。

      (2)**不区分大小写的检查子串是否存在：**
      ```javascript
      /halt/i.test(str);
      ```
      - 如果字符串 `str` 中包含子串 "halt"（不区分大小写），则返回 `true`。

      (3)**匹配多个选项：**
      ```javascript
      /[Hh]alt [A-Z]/.test(str);
      ```
      - 如果字符串 `str` 中包含 "Halt A" 或 "halt A"、"Halt B" 或 "halt B" 等，返回 `true`。

   2. `search`方法：

      (1)**搜索并返回匹配的起始位置：**

      ```javascript
      'XXX abbbbbbc'.search(/ab+c/);
      ```
      - 返回 `4`，因为正则表达式 `/ab+c/` 在字符串中匹配到 "abbbbbbc"，而匹配的起始位置是索引 `4`。

      (2)**搜索失败返回 `-1`：**
      ```javascript
      'XXX ac'.search(/ab+c/);
      ```
      - 返回 `-1`，因为正则表达式 `/ab+c/` 在字符串中没有匹配。

      (3)**灵活匹配：**
      ```javascript
      'XXX ac'.search(/ab*c/);
      ```
      - 返回 `4`，因为正则表达式 `/ab*c/` 在字符串中匹配到 "ac"，而匹配的起始位置是索引 `4`。

      (4)**匹配非数字字符：**
      ```javascript
      '12e34'.search(/[^\d]/);
      ```
      - 返回 `2`，因为正则表达式 `/[^\d]/` 匹配到字符串中的非数字字符 "e"，而匹配的起始位置是索引 `2`。

      (5)**复杂模式匹配：**

      

      ```javascript
      'foo: bar;'.search(/...\s*:\s*...\s*;/);
      ```

      - 返回 `0`，因为正则表达式 `/...\s*:\s*...\s*;/` 匹配到字符串的开头，表示以任意字符（三个点）开头，然后是冒号、零或多个空白字符、再是任意字符，最后是零或多个空白字符，以分号结尾。

   3. **`exec` 方法：**

         - `exec` 方法用于在字符串中执行正则表达式匹配，返回匹配结果的详细信息。

         ```javascript
         var pattern = /ab+c/;
         var result = pattern.exec('XXX abbbbbbc');
         
         console.log(result[0]);   // 输出匹配到的整个子串 "abbbbbbc"
         console.log(result.index); // 输出匹配的起始位置 4
         ```

   4.   **`match` 方法：**

         - `match` 方法用于在字符串中查找正则表达式匹配，返回匹配的结果数组。

         ```javascript
         var result = 'XXX abbbbbbc'.match(/ab+c/);
         
         console.log(result[0]); // 输出匹配到的整个子串 "abbbbbbc"
         ```

   5.  **`replace` 方法：**

         - `replace` 方法用于替换字符串中的匹配部分。

         ```javascript
         var newStr = '12e34'.replace(/[^\d]/, 'X');
         
         console.log(newStr); // 输出替换非数字字符后的字符串 "12X34"
         ```

      这些方法提供了更灵活的正则表达式操作。请注意，`exec` 和 `match` 方法返回的结果都是数组，其中第一个元素是匹配到的字符串，后续元素可能是分组匹配的结果。`replace` 方法则用指定的字符串替换匹配到的部分。

6. **异常错误处理**

   - 在 JavaScript 中，异常（Exceptions）是一种错误处理机制，常常用于报告和处理运行时错误。异常通常通过 `try` 和 `catch` 语句来捕获和处理。以下是一个异常处理的例子：

     ```javascript
     try {
        nonExistentFunction(); // 调用一个不存在的函数
     } catch (err) {
        console.log("Error calling function:", err.name, err.message);
     }
     ```

     在这个例子中，我们尝试调用一个名为 `nonExistentFunction` 的函数，但实际上它不存在。由于这是一个错误，JavaScript 引擎会抛出一个异常。在 `try` 语句块中，我们将有可能抛出异常的代码包裹起来。

     如果异常被抛出，程序控制流将转到 `catch` 语句块。在 `catch` 语句中，我们可以访问到异常对象 `err`，它包含了有关异常的信息，如名称（name）和消息（message）等。

     - `err.name`: 异常的名称，例如 "ReferenceError"。
     - `err.message`: 异常的详细描述，例如 "nonExistentFunction is not defined"。

     通过使用 `try` 和 `catch`，我们可以避免程序因为错误而崩溃，而是能够更加优雅地处理这些错误情况。异常会沿着调用堆栈（call stack）向上传递，直到找到匹配的 `catch` 语句为止。

     此外，`throw` 语句和 `try...catch` 结构密切相关，它们一起构成 JavaScript 中的异常处理机制。让我们通过一个整体的例子来说明它们之间的关系：

     ```javascript
     try {
        // 可能抛出异常的代码块
        throw "Help! Something went wrong.";
     } catch (err) {
        // 捕捉并处理异常的代码块
        console.log("Got Error:", err.stack || err.message || err);
     } finally {
        // 无论是否发生异常都会执行的代码块
        console.log('This block is executed after try/catch.');
     }
     ```

     在这个例子中，`try` 语句块包含了可能抛出异常的代码。如果在 `try` 语句块中发生了异常，程序流会跳到 `catch` 语句块中，其中的参数 `err` 将接收到抛出的异常值。在 `catch` 语句块中，我们可以处理异常，比如打印错误信息。

     最后，`finally` 语句块中的代码无论是否有异常都会被执行。这个块通常用于进行清理工作，比如释放资源。

     总体来说，`try` 用于包裹可能发生异常的代码块，`throw` 用于抛出异常，而 `catch` 用于捕获并处理异常，`finally` 用于确保无论是否发生异常都执行的代码块。

     

7. **引入网页：**

   - 将 JavaScript 引入网页有两种常见的方式：通过包含外部文件和直接内嵌在 HTML 中。以下是这两种方法的例子：

     **通过包含外部文件：**

     在 HTML 文件中使用 `<script>` 标签引入外部 JavaScript 文件：

     ```html
     <script type="text/javascript" src="code.js"></script>
     ```

     在这个例子中，`src` 属性指定了外部 JavaScript 文件的路径，浏览器会加载并执行这个文件。这是一种组织代码的好方法，可以将 JavaScript 代码分离到单独的文件中，使得代码更易于维护和管理。

     **内嵌在 HTML 中：**

     直接在 HTML 文件中使用 `<script>` 标签内嵌 JavaScript 代码：

     ```html
     <script type="text/javascript">
     //<![CDATA[
         // JavaScript 代码写在这里...
     //]]>
     </script>
     ```

     在这个例子中，JavaScript 代码直接嵌入到 HTML 文件中。这种方式适用于较小的脚本或在页面中只使用一次的情况。需要注意的是，在 `<script>` 标签中可以包含 `//<![CDATA[` 和 `//]]>` 注释，这是为了确保在 XHTML 文档中正确处理脚本。

     无论是外部文件还是内嵌代码，都可以在 HTML 文档中的 `<head>` 或 `<body>` 部分使用。选择使用哪种方式通常取决于项目的组织结构和需求。

     总体来说，这两种方式都是将 JavaScript 引入网页的有效方法，开发者可以根据实际需求选择适合的方式。

8.   **`this` 关键字：**

   - 在 JavaScript 中，`this` 关键字指向当前执行上下文中的对象。然而，在不同的情境下，`this` 的值可能会有所不同。

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>This Keyword Example</title>
   </head>
   <body>
   
   <script type="text/javascript">
       // 在全局作用域中，this 指向全局对象（浏览器中通常是 window）
       console.log(this);
   
       function myFunction() {
           // 在函数内部，this 的值取决于调用方式
           console.log(this);
       }
   
       myFunction(); // 这里的 this 取决于调用方式
   </script>
   
   </body>
   </html>
   ```

   `this` 的值在不同的上下文中有不同的表现，可以是全局对象、当前对象（当作为对象方法调用时），或者 `undefined`（在严格模式下，函数内部没有明确指定 `this` 的情况下）。

9. **继承（Inheritance）：**

   - JavaScript 使用原型链实现继承。对象可以通过原型链访问其他对象的属性和方法。


   ```javascript
   // 父类构造函数
   function Animal(name) {
       this.name = name;
   }
   
   // 父类方法
   Animal.prototype.sayHello = function () {
       console.log("Hello, I'm " + this.name);
   };
   
   // 子类构造函数
   function Dog(name, breed) {
       Animal.call(this, name); // 调用父类构造函数
       this.breed = breed;
   }
   
   // 设置子类的原型为父类的实例，建立原型链
   Dog.prototype = Object.create(Animal.prototype);
   
   // 子类方法
   Dog.prototype.bark = function () {
       console.log("Woof! I'm a " + this.breed);
   };
   
   var myDog = new Dog("Buddy", "Golden Retriever");
   myDog.sayHello(); // 调用父类方法
   myDog.bark();     // 调用子类方法
   ```

   在这个例子中，`Dog` 类继承了 `Animal` 类。通过在子类的构造函数中调用父类的构造函数（使用 `Animal.call(this, name)`），并通过 `Object.create(Animal.prototype)` 将子类的原型设置为父类的实例，建立了原型链。

10.  **箭头函数（Arrow Functions）：**

   - 箭头函数是 ES6 引入的一种新的函数语法，它有一些特性使其在某些情境下更加方便。


   ```javascript
   // 普通函数
   function add(a, b) {
       return a + b;
   }
   
   // 箭头函数
   const arrowAdd = (a, b) => a + b;
   
   console.log(add(2, 3));        // 输出 5
   console.log(arrowAdd(2, 3));   // 输出 5
   ```

   箭头函数相比普通函数有一些不同之处，其中一个主要区别是箭头函数没有自己的 `this`，它继承自外围作用域。这使得在一些情境下，特别是在回调函数中，箭头函数更加方便。

   ```javascript
   function Counter() {
       this.count = 0;
   
       // 使用箭头函数，保留了 Counter 对象的 this
       this.increment = () => {
           this.count++;
           console.log(this.count);
       };
   }
   
   const counter = new Counter();
   counter.increment(); // 输出 1
   ```

   在上面的例子中，箭头函数作为对象的方法，保留了对象 `Counter` 的 `this`。这与普通函数不同，普通函数中的 `this` 可能会受到调用方式的影响。

​		这些是JavaScript的一些基本语法元素，有助于实现网页的动态功能。可以在 [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide) 等网站上找到详细的JavaScript文档。

##### DOM

​		DOM（文档对象模型）是一种编程接口，它允许程序通过脚本语言（通常是 JavaScript）动态地访问和修改 HTML 或 XML 文档的结构、内容和样式。下面是一些关于 DOM 的基本语法以及参考手册中的信息：

1. 获取元素：

- **通过 ID 获取元素：**

  ```javascript
  var elementById = document.getElementById('elementId');
  ```

- **通过标签名获取元素集合：**

  ```javascript
  var elementsByTagName = document.getElementsByTagName('tagName');
  ```

- **通过类名获取元素集合：**

  ```javascript
  var elementsByClassName = document.getElementsByClassName('className');
  ```

- **通过选择器获取元素：**

  ```javascript
  var elementBySelector = document.querySelector('selector');
  var elementsBySelectorAll = document.querySelectorAll('selector');
  ```

2. 操作元素：

- **获取和设置元素内容：**

  ```javascript
  var element = document.getElementById('elementId');
  var content = element.innerHTML; // 获取内容
  element.innerHTML = 'New Content'; // 设置内容
  ```

- **获取和设置元素属性：**

  ```javascript
  var element = document.getElementById('elementId');
  var attribute = element.getAttribute('attributeName'); // 获取属性
  element.setAttribute('attributeName', 'New Value'); // 设置属性
  ```

3. 创建和插入元素：

- **创建新元素：**

  ```javascript
  var newElement = document.createElement('tagName');
  ```

- **插入元素到父元素中：**

  ```javascript
  var parentElement = document.getElementById('parentId');
  parentElement.appendChild(newElement);
  ```

4. 事件处理：

- **添加事件监听器：**

  ```javascript
  var element = document.getElementById('elementId');
  element.addEventListener('click', function() {
      // 处理点击事件的代码
  });
  ```

- **移除事件监听器：**

  ```javascript
  element.removeEventListener('click', eventHandlerFunction);
  ```

<center class="half">
    <img src="https://s2.loli.net/2024/01/01/kAyJKUseVZRMNwS.png" width= 500 />
    <img src="https://s2.loli.net/2024/01/01/AuKbh7ONsBvwXei.png" width= 300 />
</center>


​		在 JavaScript 中，关于 DOM 的相关信息可以在 MDN Web Docs 上找到。MDN 提供了详细而全面的文档，包括 DOM 的概述、API 参考、教程等，是学习和查阅 DOM 相关信息的良好资源。在 MDN 上，你可以找到各种 DOM 元素和方法的详细说明，了解如何使用和操作 DOM。

##### ReactJS

 React的主要构成可以分为以下几个板块：

1. **JSX（JavaScript XML）：**

- React 使用 JSX 语法来描述用户界面的结构。JSX 允许在 JavaScript 中编写类似 XML 或 HTML 的代码，使得界面的声明更加直观。

```jsx
const element = <h1>Hello, World!</h1>;
```

2. **组件（Components）：**

- React 应用由组件构成，是应用的基本单元。组件可以是函数式组件或类组件，用于封装 UI 和行为逻辑。每个组件都可以接收属性（Props）和管理内部状态（State）。

```jsx
// 函数式组件
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

// 类组件
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

3. **Props 和 State：**

- **Props（属性）：** 通过属性传递数据，是从父组件向子组件传递信息的方式。
  
  ```jsx
  // 通过属性传递数据
  <ChildComponent name="John" />
  ```

- **State（状态）：** 组件内部的可变数据，通过 `setState` 进行更新。影响组件的外观和行为。

  ```jsx
  // 组件内部状态
  class Counter extends React.Component {
    constructor(props) {
      super(props);
      this.state = { count: 0 };
    }
    // ...
  }
  ```

4. **生命周期方法（Lifecycle Methods）：**

![](CS142 Web Applications/mVk9f2Wu76Thsa3.png)

- React 组件具有生命周期方法，用于在组件的不同阶段执行逻辑，如挂载、更新和卸载。


```jsx
class Example extends React.Component {
  componentDidMount() {
    // 组件挂载后执行的逻辑
  }

  componentDidUpdate(prevProps, prevState) {
    // 组件更新后执行的逻辑
  }

  componentWillUnmount() {
    // 组件卸载前执行的逻辑
  }
  
  render() {
    return <p>Example Component</p>;
  }
}
```

5. **渲染（Rendering）：**

- React 通过虚拟 DOM 的概念，将组件的状态映射到界面上。当状态发生变化时，React 会高效地更新虚拟 DOM，并最终更新实际 DOM，以确保界面的同步渲染。

```jsx
render() {
     let label = React.createElement('label', null,'Name: ');
     let input = React.createElement('input',
         { type: 'text', value: this.state.yourName,
         onChange: (event) => this.handleChange(event) });
    
    let h1 = React.createElement('h1', null,
     'Hello ', this.state.yourName, '!');
    
     return React.createElement('div', null, label, input, h1);
}

```

6. **事件处理和交互（Event Handling and Interaction）：**

- React 支持通过事件处理函数处理用户输入和交互。通过事件处理，可以在用户与组件交互时触发相应的行为。

```jsx
class Button extends React.Component {
  handleClick() {
    // 处理按钮点击事件的逻辑
  }

  render() {
    return <button onClick={this.handleClick}>Click me</button>;
  }
}
```

7. **网页部署（ReactJS Web Application Page）**

- 整个 React 应用的逻辑和 UI 结构被打包到 `reactApp.bundle.js` 中。这个 JavaScript 文件包含了 React 组件的定义和相关功能。

```jsx
<script src="./webpackOutput/reactApp.bundle.js"></script>
```

![](CS142 Web Applications/ec6ZFUuBL4ktwsl.png)

##### SPA

​		Router 是用于处理 React 应用中导航和路由的库。它提供了一种在单页面应用中管理页面导航的方式。以下是 React Router 的基本语法概述以及参考手册中的根据：

1. **安装 React Router：**

- 首先，你需要通过 npm 或 yarn 安装 React Router：


```bash
npm install react-router-dom
# 或者
yarn add react-router-dom
```

2. **基本用法：**

- React Router 主要包括以下几个核心组件：


1. **`<BrowserRouter>`：** 使用 HTML5 history API（pushState, replaceState 和 popstate 事件）来保持 UI 和 URL 的同步。

```jsx
import { BrowserRouter as Router, Route, Link } from "react-router-dom";

function App() {
  return (
    <Router>
      <div>
        <nav>
          <ul>
            <li>
              <Link to="/">Home</Link>
            </li>
            <li>
              <Link to="/about">About</Link>
            </li>
          </ul>
        </nav>

        <Route path="/" exact component={Home} />
        <Route path="/about" component={About} />
      </div>
    </Router>
  );
}
```

1.  **`<Route>` :**

   - **`path`** 属性:** 定义匹配的 URL 路径 

   - **`component` 属性：** 指定与路径匹配时要渲染的组件。

2. **`<Link>`**:

   - **`to` 属性：** 指定链接要导航到的路径。

5. **路由参数：**

- 通过在路由路径中使用 `:parameter`，可以定义路由参数，这些参数将传递给匹配的组件。


```jsx
<Route path="/users/:id" component={User} />
```

- 在组件中通过 `props.match.params.id` 访问路由参数。


6. **嵌套路由：**

- 可以在组件内部嵌套 `<Route>`，实现嵌套路由。


```jsx
function App() {
  return (
    <Router>
      <div>
        <Route path="/" exact component={Home} />
        <Route path="/topics" component={Topics} />
      </div>
    </Router>
  );
}

function Topics() {
  return (
    <div>
      <h2>Topics</h2>
      <Route path="/topics/:topicId" component={Topic} />
    </div>
  );
}
```

7. **使用 `useHistory` 钩子：**

- React Router 提供了 `useHistory` 钩子，用于在组件中进行编程式导航。


```jsx
import { useHistory } from "react-router-dom";

function MyComponent() {
  const history = useHistory();

  function handleClick() {
    history.push("/new-route");
  }

  return <button onClick={handleClick}>Go to New Route</button>;
}
```

​		Router 的详细文档可以在 [React Router 文档](https://reactrouter.com/web/guides/quick-start) 中找到。该文档包含了更多高级用法、API 参考以及示例。建议在实际开发中参考官方文档以获取最新和详细的信息。

##### RWD

​		在实现响应式 Web 设计时，使用 CSS 媒体查询（Media Queries）是一种常见的方法。媒体查询允许你根据设备属性或浏览器窗口的特定特征应用不同的样式。以下是基本的 CSS 媒体查询语法以及参考手册中的根据：

1. **基本媒体查询语法：**

- 使用 `@media` 关键字和条件来定义媒体查询。


```css
/* 媒体查询 - 小于等于 600 像素时应用以下样式 */
@media only screen and (max-width: 600px) {
  body {
    background-color: lightblue;
  }
}

/* 媒体查询 - 大于 768 像素且小于 1024 像素时应用以下样式 */
@media only screen and (min-width: 768px) and (max-width: 1023px) {
  body {
    background-color: lightgreen;
  }
}

/* 媒体查询 - 打印时应用以下样式 */
@media print {
  body {
    color: black;
  }
}
```

2. **媒体查询属性：**

- **`width` 和 `height`：** 指定视口的宽度和高度。
- **`min-width` 和 `max-width`：** 设置视口宽度的最小和最大值。
- **`orientation`：** 检测设备是横向还是纵向。

**实例演示：**

```css
/* 移动设备上的样式 */
@media only screen and (max-width: 767px) {
  /* 在小屏幕上应用的样式 */
  body {
    font-size: 14px;
  }
}

/* 平板和桌面设备上的样式 */
@media only screen and (min-width: 768px) {
  /* 在中等和大屏幕上应用的样式 */
  body {
    font-size: 16px;
  }
}
```

- 这个示例演示了如何使用媒体查询根据屏幕宽度应用不同的字体大小。在小屏幕上，字体大小为 14px，在中等和大屏幕上为 16px。

![](https://s2.loli.net/2024/01/01/tunVh6wKlmbAqz9.png)

​		还可以在 MDN Web Docs 的 [Media Queries](https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries) 页面找到详细的媒体查询文档，其中包含了各种属性、语法示例以及解释。MDN Web Docs 是一个权威的 Web 技术文档资源，提供了广泛的 CSS 文档。

##### WebApps

​		Material-UI 是一个基于 React 的 UI 组件库，它实现了 Material Design 规范。以下是 Material-UI 的基本语法概述以及参考手册中的根据：

1. **安装 Material-UI：**

- 首先，你需要通过 npm 或 yarn 安装 Material-UI：


```bash
npm install @mui/material @emotion/react @emotion/styled
# 或者
yarn add @mui/material @emotion/react @emotion/styled
```

2. **导入组件：**

- Material-UI 的组件可以通过以下方式导入：


```jsx
import Button from '@mui/material/Button';
import TextField from '@mui/material/TextField';
import { AppBar, Toolbar, Typography } from '@mui/material';
```

3. **使用组件：**

```jsx
import React from 'react';
import Button from '@mui/material/Button';
import TextField from '@mui/material/TextField';

function MyComponent() {
  return (
    <div>
      <Button variant="contained" color="primary">
        Primary Button
      </Button>
      <TextField label="Enter Text" variant="outlined" />
    </div>
  );
}
```

4. **Material-UI 主题定制：**

- Material-UI 允许你通过 `ThemeProvider` 来定制主题，以适应你的应用程序设计。


```jsx
import { createTheme, ThemeProvider } from '@mui/material/styles';

const theme = createTheme();

function App() {
  return (
    <ThemeProvider theme={theme}>
      {/* Your components here */}
    </ThemeProvider>
  );
}
```

5. **响应式设计：**

- Material-UI 提供了一些响应式设计的组件，比如 `Hidden`，可以根据屏幕尺寸隐藏或显示内容。


```jsx
import Hidden from '@mui/material/Hidden';

function ResponsiveComponent() {
  return (
    <div>
      <Hidden mdUp>
        {/* 在中等及以下屏幕上隐藏 */}
        <p>Hidden on medium and below screens</p>
      </Hidden>
      <Hidden smDown>
        {/* 在小屏幕及以上屏幕上隐藏 */}
        <p>Hidden on small and above screens</p>
      </Hidden>
    </div>
  );
}
```

![](CS142 Web Applications/kpF4loVDETYB5Gv.png)		

​		Material-UI 的详细文档可以在 [Material-UI Documentation](https://mui.com/) 中找到。该文档包含了丰富的示例、API 参考和主题定制指南。建议在实际开发中参考官方文档以获取最新和详细的信息。

##### HTTP

​		HTTP（Hypertext Transfer Protocol）是一种用于传输超文本的应用层协议。以下是 HTTP 的基本语法概述以及参考手册中的根据：

1. **HTTP 请求方法：**
   - **GET：** 从服务器获取资源。
   - **POST：** 向服务器提交数据，用于创建新资源。
   - **PUT：** 向服务器提交数据，用于更新资源。
   - **DELETE：** 从服务器删除资源。
   - **HEAD：** 与 GET 方法类似，但服务器只返回头部信息，不返回实体的主体部分

2. **HTTP 请求结构：**

- HTTP 请求由请求行、请求头部和请求体组成。


```plaintext
GET /path/to/resource HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:100.0) Gecko/20100101 Firefox/100.0
```

- **请求行（Request Line）：** 包含请求方法、请求的资源路径和协议版本。
- **请求头部（Request Headers）：** 包含关于请求的附加信息，如 `Host`、`User-Agent` 等。
- **请求体（Request Body）：** 对于 POST 或 PUT 等包含请求数据的方法，请求体包含实际的数据。

3. **HTTP 响应结构：**

- HTTP 响应由状态行、响应头部和响应体组成。


```plaintext
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1234

<!DOCTYPE html>
<html>
<head>
  <title>Example Page</title>
</head>
<body>
  <h1>Hello, World!</h1>
</body>
</html>
```

- **状态行（Status Line）：** 包含协议版本、状态码和状态描述。
- **响应头部（Response Headers）：** 包含关于响应的附加信息，如 `Content-Type`、`Content-Length` 等。
- **响应体（Response Body）：** 包含实际的响应数据，如 HTML 内容、JSON 数据等。

​		HTTP 协议的详细规范可以在 [Hypertext Transfer Protocol (HTTP) - IETF](https://datatracker.ietf.org/doc/html/rfc7230) 找到。这个文档定义了 HTTP 协议的基本语法、状态码、头部字段等详细信息。在实际开发中，可以根据这个标准来理解和实现 HTTP 通信。

##### ServerCom

​		AJAX（Asynchronous JavaScript and XML）是一种用于创建交互式网页应用的技术。它通过在后台与服务器进行异步通信，实现了页面无需刷新就能够更新部分内容的能力。虽然 AJAX 的名称中包含 "XML"，但实际上，它的数据交换格式并不限于 XML，可以使用 JSON 等其他格式。以下是关于 AJAX 的基本概念和用法：

1. **XMLHttpRequest 对象：**

- AJAX 最早使用的是 `XMLHttpRequest` 对象，它是浏览器提供的 API，用于在前端发起 HTTP 请求。


```javascript
var xhr = new XMLHttpRequest();
```

2. **基本用法：**

- **GET 请求：**

```javascript
xhr.open('GET', 'https://api.example.com/data', true);
xhr.send();
```

- **POST 请求：**

```javascript
xhr.open('POST', 'https://api.example.com/submit', true);
xhr.setRequestHeader('Content-Type', 'application/json');
var data = JSON.stringify({ key: 'value' });
xhr.send(data);
```

3. **处理响应：**

- 可以通过监听 `onload` 事件来处理请求成功的情况，以及 `onerror` 事件来处理请求失败的情况。


```javascript
xhr.onload = function () {
  if (xhr.status >= 200 && xhr.status < 300) {
    console.log('Success:', xhr.responseText);
  } else {
    console.error('Request failed with status:', xhr.status);
  }
};

xhr.onerror = function () {
  console.error('Network request failed');
};
```

4. **Ready State 和状态码：**

- `XMLHttpRequest` 对象有一个 `readyState` 属性，表示请求的状态。常用的状态有：
  - `0`: 未初始化 - 对象已创建，但尚未初始化（尚未调用 `open` 方法）。
  - `1`: 打开 - 已调用 `open` 方法，但尚未调用 `send` 方法。
  - `2`: 发送 - 已调用 `send` 方法，但尚未接收到响应。
  - `3`: 接收 - 已接收到部分响应数据。
  - `4`: 完成 - 已接收到全部响应数据，可以在客户端使用。

5. **跨域请求：**

- 同 `XMLHttpRequest` 一样，AJAX 的跨域请求受到同源策略的限制，需要服务器支持 CORS 或使用其他跨域解决方案。


6. **Promise 和 Fetch API：**

- 随着时间的推移，使用 Promise 和 Fetch API 替代了传统的 AJAX。`fetch` 提供了更简洁、强大且易用的接口来处理网络请求。


```javascript
fetch('https://api.example.com/data')
  .then(response => {
    if (!response.ok) {
      throw new Error('Network request failed');
    }
    return response.json();
  })
  .then(data => console.log('Success:', data))
  .catch(error => console.error('Error:', error));
```

7. **库和框架：**

- 为了简化 AJAX 的使用，很多 JavaScript 库和框架提供了更高级的接口，例如 jQuery 的 `$.ajax` 方法、Axios 等。


​		AJAX 技术在 Web 开发中扮演了重要角色，使得前端能够通过异步请求实现更流畅的用户体验。然而，在现代开发中，推荐使用基于 Promise 的 `fetch` API 或其他现代框架提供的工具来处理网络请求。

##### NodeJS

​		Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行时，用于构建服务器端应用。它使得 JavaScript 可以在服务器端运行，并且具有事件驱动、非阻塞 I/O 操作的特性。以下是关于 Node.js 的一些基本概念和特点：

1. **事件驱动和非阻塞 I/O：**

- Node.js 基于事件驱动的模型，通过事件循环处理请求。它采用非阻塞 I/O 操作，可以在进行 I/O 操作时继续执行后续代码，而不需要等待操作完成。

```javascript
let fs = require("fs"); // require is a Node module call
 // fs object wraps OS sync file system calls
// OS read() is synchronous but Node's fs.readFile is asynchronous

fs.readFile("smallFile", readDoneCallback); // Start read
function readDoneCallback(error, dataBuffer) {
 // Node callback convention: First argument is JavaScript Error object
 // dataBuffer is a special Node Buffer object
    
 if (!error) {
 console.log("smallFile contents", dataBuffer.toString());
 }
}
```

2. **单线程：**

- Node.js 是单线程的，但通过事件循环和回调函数，能够处理大量并发请求。这是通过在后台使用多线程的方式来实现的。


3. **NPM（Node Package Manager）：**

- NPM 是 Node.js 的包管理工具，用于安装、共享和管理 JavaScript 模块。开发者可以使用 NPM 安装第三方模块，也可以将自己的模块发布到 NPM。


4. **模块化：**

- Node.js 支持模块化编程，通过 `require` 关键字引入模块。每个文件都被视为一个模块，可以通过 `module.exports` 导出功能。


```javascript
// 模块文件 example.js
const myFunction = () => {
  console.log('Hello from Node.js');
};

module.exports = myFunction;

// 在另一个文件中引入模块
const myModule = require('./example');
myModule(); // 调用模块中的函数
```

5. **内置模块：**

- Node.js 提供了一系列内置模块，如 `fs`（文件系统）、`http`（HTTP 服务器和客户端）、`path`（路径操作）等，可以直接使用。


```javascript
const fs = require('fs');
fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log(data);
});
```

![](CS142 Web Applications/XqPR42duHANnYgT.png)

6. **Express 框架：**

- Express 是一个流行的 Node.js 框架，简化了构建 Web 应用的过程。它提供了路由、中间件等功能，使得开发者可以更轻松地构建 Web 服务器和 RESTful API。


```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello, Express!');
});

app.listen(3000, () => {
  console.log('Server is running on port 3000');
});
```

7. **异步编程和回调函数：**

- 由于 Node.js 的非阻塞特性，异步编程是其核心。大量的 Node.js API 是基于回调函数的，但随着异步编程的发展，Promise 和 async/await 等方式也被广泛应用。

```javascript
var fileContents = {};
async.each(['f1','f2','f3'], readIt, function (err) {
 if (!err) console.log('Done'); // fileContents filled in
 if (err) console.error('Got an error:', err.message);
});
function readIt(fileName, callback) {
 fs.readFile(fileName, function (error, dataBuffer) {
 fileContents[fileName] = dataBuffer;
 callback(error);
 });
}
```

8. **WebSocket 和实时应用：**

- Node.js 适用于构建实时应用，例如聊天应用、实时通知等，它支持 WebSocket 技术，使得双向通信更为简单。


​		Node.js 在现代 Web 开发中有着广泛的应用，特别适用于构建高性能的、事件驱动的服务器端应用。由于其轻量、高效的特性，成为前端开发者进行全栈开发的常用选择。

##### Express

​		Express 是一个流行的 Node.js Web 应用框架，它简化了构建 Web 应用和 RESTful API 的过程。以下是关于 Express 框架的一些基本概念和特点：

1. **快速搭建 Web 服务器：**

- 使用 Express 可以很容易地创建一个基本的 Web 服务器。


```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello, Express!');
});

app.listen(3000, () => {
  console.log('Server is running on port 3000');
});
```

2. **路由：**

- Express 提供了简单而灵活的路由系统，可以根据不同的 HTTP 方法和 URL 进行路由匹配。


```javascript
app.get('/users', (req, res) => {
  res.send('Get all users');
});

app.post('/users', (req, res) => {
  res.send('Create a new user');
});

app.put('/users/:id', (req, res) => {
  res.send(`Update user with ID ${req.params.id}`);
});

app.delete('/users/:id', (req, res) => {
  res.send(`Delete user with ID ${req.params.id}`);
});
```

3. **中间件：**

- Express 中的中间件是处理请求的函数，可以在请求到达路由处理之前或之后执行。它可以用于添加日志、处理身份验证、解析请求体等。


```javascript
// 日志中间件
app.use((req, res, next) => {
  console.log(`Request: ${req.method} ${req.url}`);
  next();
});

// 静态文件中间件
app.use(express.static('public'));

// 解析 JSON 请求体中间件
app.use(express.json());
```

4. **模板引擎：**

- Express 支持多种模板引擎，如 EJS、Pug（以前的 Jade）、Handlebars 等，使得在服务器端动态生成 HTML 页面更为方便。


```javascript
// 使用 EJS 模板引擎
app.set('view engine', 'ejs');

app.get('/example', (req, res) => {
  res.render('example', { title: 'Express Example' });
});
```

5. **错误处理：**

- Express 提供了简单的错误处理机制，可以通过定义错误处理中间件来处理应用程序中发生的错误。


```javascript
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Something went wrong!');
});
```

6. **RESTful API：**

- Express 很适合构建 RESTful API，通过合理的路由设计和中间件的使用，可以方便地创建符合 RESTful 风格的 API。


```javascript
// 获取所有用户
app.get('/api/users', (req, res) => {
  // 返回用户数据
});

// 获取单个用户
app.get('/api/users/:id', (req, res) => {
  // 返回指定用户数据
});

// 创建新用户
app.post('/api/users', (req, res) => {
  // 处理请求体中的数据，创建新用户
});

// 更新用户信息
app.put('/api/users/:id', (req, res) => {
  // 处理请求体中的数据，更新用户信息
});

// 删除用户
app.delete('/api/users/:id', (req, res) => {
  // 删除指定用户
});
```

7. **WebSocket 和 Socket.io：**

- 虽然 Express 本身不直接支持 WebSocket，但可以通过结合使用 Socket.io 等库来实现实时通信。


​		Express 提供了许多功能强大而灵活的特性，使得开发者可以更加高效地构建 Web 应用和 API。在 Node.js 生态系统中，Express 一直是最受欢迎和广泛应用的 Web 框架之一。

##### Database

​		MongoDB 是一种 NoSQL 数据库系统，它使用文档模型存储数据，具有灵活的架构和高度的可扩展性。以下是 MongoDB 的一些基本语法，包括数据的插入、查询、更新和删除等操作：

Setup:👉 [MongoDB安装(win版本)](https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-windows/#overview)

1. **连接到 MongoDB 数据库：**

- 在使用 MongoDB 前，首先需要连接到数据库。可以使用 `mongo` 命令行工具或者在 Node.js 中使用相应的驱动程序（如 `mongodb` 包）。

  使用 `mongo` 命令行工具：

```bash
mongo
```

- 使用 Node.js 驱动程序：

```javascript
const MongoClient = require('mongodb').MongoClient;

const url = 'mongodb://localhost:27017'; // MongoDB 服务器地址
const dbName = 'mydatabase'; // 数据库名称

MongoClient.connect(url, { useNewUrlParser: true, useUnifiedTopology: true }, (err, client) => {
  if (err) throw err;

  console.log('Connected to MongoDB');
  const db = client.db(dbName);

  // 在这里进行 MongoDB 操作

  client.close();
});
```

2. **插入数据：**

```javascript
const collection = db.collection('mycollection');

// 插入单个文档
collection.insertOne({ name: 'John Doe', age: 30 }, (err, result) => {
  if (err) throw err;
  console.log('Document inserted');
});

// 插入多个文档
collection.insertMany([
  { name: 'Jane Doe', age: 25 },
  { name: 'Bob Smith', age: 40 }
], (err, result) => {
  if (err) throw err;
  console.log('Documents inserted:', result.insertedCount);
});
```

3. **查询数据：**

```javascript
// 查询所有文档
collection.find({}).toArray((err, documents) => {
  if (err) throw err;
  console.log('All documents:', documents);
});

// 根据条件查询单个文档
collection.findOne({ name: 'John Doe' }, (err, document) => {
  if (err) throw err;
  console.log('Found document:', document);
});
```

4. **更新数据：**

```javascript
// 更新单个文档
collection.updateOne({ name: 'John Doe' }, { $set: { age: 31 } }, (err, result) => {
  if (err) throw err;
  console.log('Document updated');
});

// 更新多个文档
collection.updateMany({ age: { $lt: 30 } }, { $inc: { age: 1 } }, (err, result) => {
  if (err) throw err;
  console.log('Documents updated:', result.modifiedCount);
});
```

5. **删除数据：**

```javascript
// 删除单个文档
collection.deleteOne({ name: 'John Doe' }, (err, result) => {
  if (err) throw err;
  console.log('Document deleted');
});

// 删除多个文档
collection.deleteMany({ age: { $gte: 40 } }, (err, result) => {
  if (err) throw err;
  console.log('Documents deleted:', result.deletedCount);
});
```

6. **索引：**

- 可以通过创建索引来提高查询性能。在 MongoDB 中，可以使用 `createIndex` 方法。


```javascript
collection.createIndex({ name: 1 }, (err, result) => {
  if (err) throw err;
  console.log('Index created');
});
```

​		MongoDB 的查询语法和操作符非常灵活，可以满足各种复杂的查询和数据操作需求。

##### Sessions

​		在 Web 开发中，使用 Cookies 来调整 Session 通常指的是通过设置和处理 Cookies 来调整用户的会话状态。这包括了使用 Cookies 存储一些信息，以实现会话跟踪、用户认证等功能。以下是一些基本的用法和相关考虑：

1. **存储 Session ID：**

- 最常见的用法是将用户的 Session ID 存储在 Cookie 中。服务器端会为每个用户分配一个唯一的 Session ID，并将其设置为用户的 Cookie。这样，在用户的每次请求中，浏览器都会自动发送这个 Session ID，从而服务器能够识别用户的会话状态。


```http
Set-Cookie: sessionID=abc123; Path=/; HttpOnly; Secure
```

2. **会话跟踪：**

- 通过 Cookies 来调整 Session 可以实现会话跟踪，即使用户关闭浏览器后再次访问网站仍能保持之前的会话状态。


```javascript
// 服务器端伪代码
if (request.cookies.sessionID) {
  // 根据 sessionID 恢复用户的会话状态
} else {
  // 创建新的 sessionID，并设置到用户的 Cookie 中
  const newSessionID = generateUniqueID();
  response.setCookie('sessionID', newSessionID);
}
```

3. **用户认证：**

- Cookies 也常用于存储用户认证信息，使得用户在一段时间内无需重新登录。


```javascript
// 用户登录成功后，服务器端设置认证信息到 Cookies 中
response.setCookie('userId', '123', { maxAge: 86400000 }); // 设置过期时间为一天

// 在每次请求中，服务器端通过 Cookies 中的认证信息来验证用户身份
if (request.cookies.userId) {
  const userId = request.cookies.userId;
  // 根据 userId 获取用户信息
} else {
  // 用户未登录，需要进行登录操作
}
```

4. **安全性考虑：**

- **HttpOnly 属性：** 通过设置 `HttpOnly` 属性，可以防止客户端脚本通过 `document.cookie` 来访问 Cookie，提高安全性。

- **Secure 属性：** 对于包含敏感信息的 Cookies，可以设置 `Secure` 属性，仅在通过 HTTPS 连接时才会发送。

```http
Set-Cookie: sessionId=abc123; Path=/; HttpOnly; Secure
```

5. **过期时间：**

- Cookies 可以设置过期时间，以控制其有效期。


```javascript
// 设置 Cookies 过期时间为一天
response.setCookie('sessionId', 'abc123', { maxAge: 86400000 });
```

总体而言，使用 Cookies 来调整 Session 是一种常见而有效的做法，但需要注意安全性和隐私性的问题，以及合理设置过期时间，避免潜在的安全风险。在实际开发中，可以使用各种 Web 开发框架或库来方便地处理 Cookies 相关的操作。

##### Input

​		表单验证（Validation）是确保用户输入数据符合特定规则或条件的过程。这个过程的目的是防止无效或恶意数据进入应用程序，以确保数据的正确性、完整性和安全性。在 Web 开发中，表单验证通常由前端（客户端）和后端（服务器端）两部分组成。

1. 前端表单验证：

- 前端验证主要是通过 JavaScript 在用户填写表单时进行实时验证，提供即时的反馈。常见的前端验证方式包括：


1. **必填字段验证：** 确保用户填写了必需的字段。

    ```html
    <input type="text" name="username" required>
    ```

2. **数据类型验证：** 确保用户输入符合预期的数据类型，例如数字、邮箱地址等。

    ```html
    <input type="number" name="age" min="1" max="100">
    <input type="email" name="email">
    ```

3. **长度验证：** 检查用户输入的字符数是否在指定范围内。

    ```html
    <input type="text" name="password" minlength="8" maxlength="20">
    ```

4. **正则表达式验证：** 使用正则表达式检查用户输入是否匹配特定模式。

    ```html
    <input type="text" name="zip" pattern="\d{5}">
    ```

5. **密码确认验证：** 确保用户在注册时输入的密码和确认密码一致。

    ```html
    <input type="password" name="password">
    <input type="password" name="confirmPassword">
    ```

6. **自定义验证函数：** 使用 JavaScript 编写自定义验证函数。

    ```html
    <input type="text" name="customField" oninput="validateCustomField(this.value)">
    ```

2.  端表单验证：

- 后端验证是在数据提交到服务器端后进行的验证过程，确保数据的合法性和安全性。后端验证是更加严格和安全的一层，因为前端验证可以被绕过。后端验证通常涉及以下方面：


1. **字段存在性验证：** 确保提交的数据包含必需的字段。

2. **数据类型验证：** 确保提交的数据类型正确，例如确保一个数字字段确实包含数字。

3. **长度验证：** 确保文本字段的长度在合理的范围内。

4. **业务规则验证：** 检查数据是否符合特定的业务规则，例如检查用户名是否唯一。

5. **防御性编程：** 防止 SQL 注入、XSS 攻击等恶意输入。

- 在 Node.js 中，使用 Express 框架可以借助一些中间件来处理表单验证。例如，`express-validator` 中间件可以用于验证和清理用户输入数据。


```javascript
const { body, validationResult } = require('express-validator');

app.post(
  '/register',
  [
    // 验证用户名不能为空
    body('username').notEmpty(),

    // 验证密码必须包含数字和字母
    body('password').isAlphanumeric(),

    // 验证邮箱格式
    body('email').isEmail(),
  ],
  (req, res) => {
    // 处理表单数据
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }

    // 继续处理注册逻辑
    // ...
  }
);
```

- 在这个例子中，`express-validator` 中间件用于验证用户名、密码和邮箱的格式，如果验证失败，将返回错误信息。这有助于确保在处理用户提交的数据时，数据是有效和安全的。


​		总的来说，表单验证是保障应用程序数据质量和安全性的关键步骤，应该同时在前端和后端进行。前端验证提供用户友好的反馈，而后端验证是确保数据完整性和安全性的最后一道防线。

##### StateManagement

**State Management（状态管理）** 是指在应用程序中有效地管理和维护应用程序的状态，确保组件之间能够共享和同步数据。在前端开发中，特别是在使用像 React、Vue、Angular 这样的框架时，状态管理变得尤为重要，因为应用程序通常由多个组件组成，这些组件之间需要共享数据和状态。

以下是关于状态管理的一些关键概念和实践：

1. **组件级状态 vs 应用级状态：**

- **组件级状态：** 组件级状态是指存储在单个组件内部的状态，只在该组件中可用。这些状态通常用于管理组件内的局部数据和交互。

- **应用级状态：** 应用级状态是指跨多个组件共享的状态，可以存储在全局状态管理工具中。这样的状态允许不同组件之间的通信和数据共享。

2. **常见的状态管理工具：**

- **React：** 在 React 中，常用的状态管理工具包括本地组件状态（通过 `useState`），上下文 API（`React.createContext` 和 `useContext`），以及更强大的状态管理库，如 Redux。

- **Vue：** Vue 提供了响应式数据和 `Vuex` 这样的状态管理库来管理组件之间的状态。`Vuex` 提供了一个全局的状态存储，使得不同组件之间可以共享状态。

- **Angular：** Angular 使用服务（Service）来管理应用的状态。通过依赖注入，Angular 服务可以在整个应用中共享状态。

![](https://s2.loli.net/2024/01/01/1jwAkync3mbrJiq.png)

3. **Redux：**

​			Redux 是一个独立的状态管理库，主要用于 React 应用。它提供了一个单一的存储（Store）来存储整个应用的状态，通过动作（Actions）和纯函数的方式来更新状态。Redux 的核心概念包括：

- **Store：** 存储整个应用的状态。

- **Actions：** 描述状态变化的对象。

- **Reducers：** 纯函数，用于根据动作更新状态。

- **Dispatch：** 发布动作，触发状态更新。

- **Selectors：** 从状态中选择数据。

4.  **状态管理的挑战：**

- **组件通信：** 当应用规模变大时，组件之间的状态共享和通信变得复杂。状态应该在什么级别管理，以及如何在组件之间传递状态是挑战之一。

- **异步操作：** 处理异步操作和副作用可能会使状态管理变得复杂。例如，在获取数据时，需要处理异步请求和更新状态。

- **性能优化：** 在大型应用中，避免不必要的状态更新和重新渲染是关键。状态管理工具通常提供了一些优化机制，但开发人员仍然需要注意性能问题。

5. **未来趋势：**

- **React Hooks：** 使用 React Hooks，如 `useState` 和 `useReducer`，使得组件级状态管理更加灵活和简单。

- **新兴状态管理工具：** 一些新兴的状态管理工具，如 Recoil、Zustand 等，试图提供更简单和灵活的状态管理方案。

- **GraphQL：** 对于应用程序的数据层管理，GraphQL 提供了更灵活的数据查询和状态管理方式。

​		在选择状态管理工具时，开发人员应该根据应用的规模、复杂性和团队的经验来做出决策。合适的状态管理工具可以使应用更易于维护和扩展。

##### WebAppSercurity

Network Attack

Secure Sockets Layer (SSL) & Transport Layer Security (TLS) - HTTPS 

- Protocol used for secure communication between browsers and servers 
- Browser uses certificate to verify server's identity 
- Only one way: SSL/TLS does not allow the server to verify browser identity 
- Uses certificates and public-key encryption to pass a secret session-specific key from browser to server

![](CS142 Web Applications/pwWmVEz8IN1ldO3.png)

Session Attack

……

## Project

#### proj 1 HTML and CSS

Create a *single* HTML document that presents two different appearances, determined by the document's CSS stylesheet.

难度：⭐

HTML的基本框架以及导入CSS的样式，同时设计简单的CSS风格

贴一段Style A的代码：

```css
html {
  /* 设置整个 HTML 元素的高度为100%。确保后续元素能够相对于 HTML 元素的高度进行百分比布局 */
  height: 100%;
}

body {
  /* display: inline-block; */
  /* height: 100%; */
  white-space: nowrap; /* 防止文本换行，即在一行内显示，不自动换行 */
  height: 100%;
  display: flex; /* 使用 Flex 布局 */
  flex-direction: column; /* 设置主轴方向为垂直方向，让子元素纵向排列 */
  align-items: center; /* 在交叉轴上居中对齐子元素 */
  justify-content: space-evenly; /* 在主轴上将子元素均匀分布，使它们在主轴上的间距相等且居中 */
}

div {
  width: 100px; /* px:像素 */
  height: 100px;
  border-top: 1px solid #687291; /* 在顶部添加1像素的实线边框，颜色为 #687291 */
  font: 40px Tahoma, Arial, sans-serif; /* 设置字体大小和字体系列 */
  display: flex;
  justify-content: center;
  flex: none; /* 禁止子元素的伸缩，即它们不会随着 Flex 容器的伸缩而伸缩 */
}

div:nth-child(even) {
  /* 选择偶数位置的 <div> 元素，并设置其样式 */
  background-color: #dfe1e7;
}

div:nth-child(odd) {
  background-color: #eeeff2;
}

div:nth-last-child(1) {
  border: 4px solid black; /* 设置4像素宽的黑色实线边框 */
  background-color: #687291;
  align-items: center;
}

```

#### proj 2 JavaScript Calisthenics

P1: Generate a Multi Filter Function;

P2: Template Processor

P3: Fix the file to not pollute the global namespace

难度：⭐

P1利用Array自带的filter函数即可，P2则是利用正则表达式和match函数进行了文本替换，P3使用的是自执行函数（IIFE）的模式，用括号包裹住函数来创建一个独立的作用域，防止变量污染全局作用域。

```javascript
/* Create function classes and update internal variables */
function Cs142TemplateProcessor(template) {
    this.template = template;
}	

Cs142TemplateProcessor.prototype.fillIn = function(dictionary) {
    /* Setting regular expressions for filtering */
    let filledtemplate = this.template;
    const re = /{{([^{]*)}}/g;
    const matches = filledtemplate.match(re);
	
    if (matches) {
        matches.forEach(placeholder => {
            /* Perform text substitution (invalid are left behind */)
            const property = placeholder.replace("{{", "").replace("}}", "");
            if (dictionary[property] !== undefined) {
                filledtemplate = filledtemplate.replace(placeholder, dictionary[property]);
            } else {
                filledtemplate = filledtemplate.replace(placeholder, "");
            }
        });
    }

    return filledtemplate;
}
```

#### proj 3 JavaScript and DOM

P1: JavaScript Date Picker

P2: Simple Table Template Processing

难度：⭐⭐

P1是利用DOM的特性进行table的填充，首先利用`document.createElement`函数在网页上生成新的元素，再通过`appendChild`函数在元素中添加新的子元素。特别的，利用`Date`这个内置类可以很好地解析input的`id`，以及对于`row`，`head`和`cell`等元素，善用`insertCell`函数。

P2则是综合了P1与proj 2，先利用DOM创建table，再利用`Cs142TemplateProcessor`进行文本的替代（别忘了在最后加上`table.style.visibility = "visible";`，否则由于DOM的状态渲染不是实时更新的所以会出现一片空白）

```javascript
"use strict";

class TableTemplate{
    static fillIn(id, dict, columnName){
        // process the header
        const table = document.getElementById(id);
        const rows = table.rows;
        const header = rows[0];
        const headerProcessor = new Cs142TemplateProcessor(header.innerHTML);
        header.innerHTML = headerProcessor.fillIn(dict);

        // get the specified column
        let index = 0;
        for (let i = 0; i < header.cells.length; i++) {
            const headerText = header.cells[i].innerHTML;
            if (headerText === columnName) {
                index = i;
            }
        }

        // process the columnValue
        let ele;
        for(let i = 0; i < rows.length; i++){
            // if columnName exists, substitute the value in this column;
            // if not, pass the whole row
            ele = columnName ? rows[i].cells[index] : rows[i];

            const cellProcessor = new Cs142TemplateProcessor(ele.innerHTML); // process
            ele.innerHTML = cellProcessor.fillIn(dict);
        }

        // examine the visibility property
        table.style.visibility = "visible";
    };
}
```

#### proj 4 Page Generation with ReactJS

P1: Understand and update the example view

P2: Create a new component - states view

P3: Personalizing the Layout

P4: Add dynamic switching of the views

P5: Single page app

难度：⭐⭐⭐

P1只需修改Component中的Example，在`render`函数返回的`view`开头添加一个新的`div`用于呈现motto的内容和添加`input`文本框即可。P2利用了之前缩写过的`filter`函数的逻辑对所输入数字进行了筛选，然后利用无序列表呈现出来即可。P3创建了一个新的`Header`组件，返回一个`<header>` `</header>`包裹的标题文本即可。

P4需要处理在`Exampl`和`States`两个组件之间切换的问题，核心逻辑是在选择组件渲染前进行判断，由于组件的数据源不同，记得在`p4.html`中导入两个`modealdata`的文件（笔者在这里困了好久，一直以为是组件的问题）。P5引入了`Route`，注意写法就行，和P4基本一致。

```jsx
/* reaction of the button */ 	
	handleViewChange = () => {	this.setState({view: !this.state.view}); }
/* render the chosen component */
    render () {
        return (
            <div>
                <button id='dynamic-button'
                        onClick={this.handleViewChange}>
                    {`Switch to ${this.state.view ? EXAMPLE : STATE} view`}
                </button>
                {this.state.view ? <Example /> : <States />}
            </div>
        );
    }
```

```jsx
/* import the Router to direct the page */		
		<HashRouter>
        	<Link to="/states"><button>Switch to States</button></Link>
            <Link to="/example"><button>Switch to Example</button></Link>
		
            <Route path="/states" component={States} /> 
            <Route path="/example" component={Example} />
            <Route path="/" render={() => <h1>Welcome!</h1>} />
        </HashRouter>
/* 'to' in link should match the 'path' in Route */
```

#### proj 5 Single Page Applications

p1: Create the Photo Sharing Application

p2: Fetch model data from the web server

难度：⭐⭐⭐（proj 5是重中之重，难度相较于前面3个有显著提高，并且后面的proj都会用到）

P1需要完成4个基本组件的构建，基本逻辑并不难，主要是熟悉`Material-UI`的格式（比如`<Typography>`）以及记住如何创造组件的生命周期函数（如`componentDidMount`），tips: 学会利用`map()`和`match()`函数会简洁很多。P2主要任务是将P1中从`photoApp.js`里获取的数据改为从`url("http://localhost:3000/test/info" )`等网址中获取，难点在于设计`fetchModel()`函数，笔者使用的是`XMLHttPRequest()`的方式向`web server`发送请求，再对返回的`JSON`对象进行解析来获取目标网站的数据，遗憾的是该网址的测试用例已经不对外开放了，所以最后只剩孤零零的`loading...`

```javascript
function fetchModel(url) {
        return new Promise(function(resolve, reject) {
            console.log("Fetching data from: ", url);

            // Load via AJAX
            const request = new XMLHttpRequest();
            request.onreadystatechange = function() {
                // a function to be executed whenever the readyState changes.
                if (this.readyState===4 && this.status===200) {
                    // On Success return:
                    resolve({ data: JSON.parse(this.responseText) });
                }
                // On Error return:
                this.onerror = () => setTimeout(()=>reject(new Error({ status: this.status, statusText: this.statusText })), 0);
            };

            // Send request to server, the path is url variable
            request.open("GET", url, true);
            // Start the request sending
            request.send();
        });
    }
```

#### proj 6 Appserver and Database

p1: Convert the web server to use the database

p2: Convert your app to use `axios`

难度：⭐⭐⭐

P1需要利用`mongoose`的方法将原本从`cs142models`中获取的数据改为从`mongodb`的数据库中获取，注意利用`app,get()`的方式获取时要匹配不同id的数据，以及过程中的`parse`与`log`。P2则是需要对于每个组件将`fetchModel()`函数获取的格式统一换成利用`axios`的异步获取格式，注意`.then()`和`.catch()`的异常处理即可。

```javascript
/*
 * URL /user/:id - Return the information for User (id)
 */
app.get('/user/:id', function (request, response) {
    const id = request.params.id;
    /**
     * Finding a single user from user's ID
     */
    User.findOne({_id: id}, function(err, user){
        if (err) {
            console.log(`** User ${id}: Not Found! **`);
            response.status(400).send(JSON.stringify(err));
        } else {
            console.log(`** Read server path /user/${id} Success! **`);
            const userObj = JSON.parse(JSON.stringify(user)); 
            								// convert mongoose data to JS data
            delete userObj.__v;             // remove unnecessary property
            response.json(userObj);
        }
    });
});
```

#### proj 7 Sessions and Input

P1: Simple Login

P2: New Comments 

P3: Photo Uploading

P4: Registration and Passwords

难度：⭐⭐⭐

P1需要创建一个简单的登陆机制，创建新组件`LoginRegister`来从数据库中验证登录信息，同时在`webServer.js`中利用`app.post()`的新方法来更新服务器状态。P2则是需要在`webServer.js`添加对`/commentsOfPhoto/:photo_id`的匹配，在根据登录信息添加新对象储存`comment`。P3与P2相似，需要利用`Photo`的对象通过`mongoose`向数据库中添加新的图片，注意每一次异步操作的异常处理。P4需要创建新的注册机制，利用已有的组件`LoginRegister`，在其中使用`axios`发送包含登录信息的 POST 请求。

```jsx
handleRegisterSubmit = e => {
    e.preventDefault();

    // make sure the two passwords entered are the same
    if (this.state.newPassword !== this.state.newPassword2) {
      this.setState({ registeredMessage: "The two passwords are NOT the same, please try again" }); // update message prompt to indicate user
      return;
    }
    const newUser = this.getNewUser();
    console.log(newUser);
    
    // axios to send a POST request with login info
    axios
      .post("/user", newUser) // POST request sent!
      .then(response => {   // Handle success
        console.log(`** LoginRegister: new User register Success! **`); 
        this.setState({ registeredMessage: response.data.message }); // update message prompt to indicate user
      })
      .catch(error => {     // Handle error
        console.log(`** LoginRegister: new User loggin Fail! **`); 
        this.setState({ registeredMessage: error.response.data.message }); // update error prompt to indicate user
      });
  };
```

#### proj 8  Photo App Sprint

难度：⭐⭐⭐⭐⭐

# 感想(Summary)

​		在整个回顾的过程中，笔者的思路也一步步重新呈现，同时也处理掉了一些之前侥幸逃过一劫的bug，对于Full Stack Development的理解也更深了一层。CS142对于前端JS+CSS+html的解释让笔者慢慢开始发现日常生活中多种多样网页的精妙绝伦之处，当自己第一次按下陌生的`F12`，看到Google弹出的全新界面，那种发现新大陆的兴奋与好奇也许正是我们这些普普通通的计算机人所应该秉持的初心吧。

​		最后，一门课的终了不过是汪洋大海中汇入的一股涓涓细流，接下来的路对笔者还很长很长，希望自己能够不被生活和学业的压力所裹挟，永远葆有那股冲动和新奇，成为那股吹彻山野、不被定义的风。

以我敬爱的jyy老师的一句话作结，自勉，同时也送给各位：

>​		我们都是活生生的人, 从小就被不由自主地教导用最小的付出获得最大的得到, 经常会忘记我们究竟要的是什么. 我承认我完美主义, 但我想每个人心中都有那一份求知的渴望和对真理的向往, "大学"的灵魂也就在于超越世俗, 超越时代的纯真和理想 -- 我们不是要讨好企业的毕业生, 而是要寻找改变世界的力量.									——jyy
>

# 附录(Appendix)

[CS142(22Sp)课程官网](https://web.stanford.edu/class/archive/cs/cs142/cs142.1226/lectures.html)

[ESLint docs](https://eslint.org/docs/latest/)

[MDN web docs](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flexible_Box_Layout)

[React手册](https://react.dev/reference/react)

[Material UI手册](https://mui.com/material-ui/getting-started/)

[MongoDB手册](https://www.mongodb.com/docs/manual/introduction/)