# sharing_meeting.github.io
# 浏览器是怎么工作的

## 浏览器的主要组件

- 用户界面
- 网络
- 呈现引擎(Rendering Engine 渲染引擎)
- JavaScript解释器
- 数据存储

## 内核

浏览器内核由渲染引擎和JS引擎组成，不同的浏览器、即使同一浏览器不同型号可能渲染引擎和JS引擎都不一样。
最开始渲染引擎和 JS 引擎并没有区分的很明确，后来 JS 引擎越来越独立，内核就倾向于只指渲染引擎。
![webkit结构](./image/webkit.png)

## 呈现引擎

-chromium

- Chromium - Blink - Chrome
- Webkit - WebCore - Safari
- Trident - IE
- Gecko - Firefox
- Presto - Opera(现在用Blink)

把 HTML 和 CSS 代码转换成画面。

### 主流程

呈现引擎一开始会从网络层获取请求文档的内容，内容的大小一般限制在 8000 个块以内。

呈现引擎将开始解析 HTML 文档，并将各标记逐个转化成“内容树”上的 DOM 节点。同时也会解析外部 CSS 文件以及样式元素中的样式数据。HTML 中这些带有视觉指令的样式信息将用于创建另一个树结构：呈现树。

呈现树包含多个带有视觉属性（如颜色和尺寸）的矩形（呈现器）。这些矩形的排列顺序就是它们将在屏幕上显示的顺序。

呈现树构建完毕之后，进入“布局”处理阶段，也就是为每个节点分配一个应出现在屏幕上的确切坐标。

下一个阶段是绘制 - 呈现引擎会遍历呈现树，由用户界面后端层将每个节点绘制出来。

需要着重指出的是，这是一个渐进的过程。为达到更好的用户体验，呈现引擎会力求尽快将内容显示在屏幕上。它不必等到整个 HTML 文档解析完毕之后，就会开始构建呈现树和设置布局。在不断接收和处理来自网络的其余内容的同时，呈现引擎会将部分内容解析并显示出来。

### HTML解析

HTML 解析器的任务是将 HTML 标记解析成解析树。

#### DOM(Document Object Model)

````javascript
<html>
<body>
<p>
    Hello World
</p>
<div>
    <img src="./image/webkit.png"/>
</div>
</body>
</html>
````

![DOM树](./image/dom-tree.png)
![浏览器中查看DOM](./image/dom-tree-console.png)

#### 容错机制

### CSS解析

- declaration 声明

![rule代码块](./image/css-rule.png)
![rule树](./image/css-rule-tree.png)
![CSSOM树](./image/cssom.png)

#### 逆向解析(效率)

````html
<div>
    <div class="box">
        <p><span>s1</span></p>
        <p><span>s2</span></p>
        <p><span>s3</span></p>
        <p><span class='red'>s4</span></p>
    </div>
</div>
````

````css
div > div.box p span.red{
color:red;
}
````

#### 优先级

- 如果声明来自于“style”属性，而不是带有选择器的规则，则记为 1，否则记为 0 (= a)
- 记为选择器中 ID 属性的个数 (= b)
- 记为选择器中其他属性和伪类的个数 (= c)
- 记为选择器中元素名称和伪元素的个数 (= d)
- 数位之间没有进制

### 呈现树

DOMTree 和 CSSOM 结合生成render tree

![DOM + CSSOM](./image/render-tree-construction.png)

在 DOM 树构建的同时，浏览器还会构建另一个树结构：呈现树。这是由可视化元素按照其显示顺序而组成的树，也是文档的可视化表示。它的作用是让您按照正确的顺序绘制内容。

Firefox 将呈现树中的元素称为“框架”。WebKit 使用的术语是呈现器或呈现对象。

#### DOM树与呈现树的关系

呈现器和DOM元素是相对应的，但不是完全一致。非可视化元素不会插入呈现树，比如`display: none`, `meta`。复杂结构元素可能会对应多个呈现器，如`select`。

`select`元素有 3 个呈现器：一个用于显示区域，一个用于下拉列表框，还有一个用于按钮。

`z-index` 控制顺序

![呈现树](./image/render-tree.png)

#### 呈现器类型

每一个呈现器都代表了一个矩形的区域，通常对应于相关节点的 CSS 框。框的类型会受到与节点相关的“display”样式属性的影响。(inline, block)

### 布局

呈现器在创建完成并添加到呈现树时，并不包含位置和大小信息。计算这些值的过程称为布局。HTML采用基于流的布局模型，意味着大多数情况下只要一次遍历就能计算出几何信息(位置，大小)，处于流中靠后位置的元素通常不会影响靠前位置元素的几何特征。

#### 流

- 文档流(普通流)
- 文本流
- 绝对定位

![文档流](./image/flow-normal.png)
![文本流](./image/flow-float.png)
![绝对定位](./image/flow-absolute.png)

#### 全量和增量

全局布局是指触发了整个呈现树范围的布局，触发原因可能包括：

影响所有呈现器的全局样式更改，例如字体大小更改。屏幕大小调整。
在某些情况下，只有一个子树进行了修改，因此无需从根节点开始布局。这适用于在本地进行更改而不影响周围元素的情况，例如在文本字段中插入文本（否则每次键盘输入都将触发从根节点开始的布局）。

### 绘制

将呈现器的内容显示在屏幕上。

在绘制阶段，系统会遍历呈现树，并调用呈现器的“paint”方法，将呈现器的内容显示在屏幕上。绘制工作是使用用户界面基础组件完成的。

z-index 呈现树控制

### 重排重绘

## JavaScript引擎

- V8 - Chrome
- JavaScriptCore - Safari
- Chakra - IE
- SpiderMonkey - Firefox
- Carakan - Opera(现在用V8)

JavaScript 解析引擎就是根据 ECMAScript 定义的语言标准来动态执行 JavaScript 字符串。

解析JS的过程分为两个阶段, 语法检查阶段和运行阶段

- 语法检查
- 词法分析 将输入分割为一个个有意义的词块(token)
- 语法分析 确定词法分析器分割出的token是如何彼此关联的
- 运行阶段
- 预解析 创建执行上下文。(作用域链，this，声明提升)
- 运行

### 垃圾回收

- 标记清除(mark-and-sweep)

垃圾收集器在运行的时候会给存储在内存中的所有变量都加上标记（当然，可以使用任何标记方式）。然后，它会去掉环境中的变量以及被环境中的变量引用的变量的标记。而在此之后再被加上标记的变量将被视为准备删除的变量，原因是环境中的变量已经无法访问到这些变量了。最后，垃圾收集器完成内存清除工作，销毁那些带标记的值并回收它们所占用的内存空间。

- 引用计数(reference counting)

引用计数的含义是跟踪记录每个值被引用的次数。

### 线程

例外： Web Worker
为了利用多核CPU的计算能力，HTML5提出Web Worker标准，允许JavaScript脚本创建多个线程，但是子线程完全受主线程控制，且不得操作DOM。所以，这个新标准并没有改变JavaScript单线程的本质。

和DOM线程互斥

### 事件循环(event loop)

异步 非阻塞

负责主线程与其他进程（主要是各种I/O操作）的通信

- 所有任务都在主线程上执行，形成一个执行栈（execution context stack）。
- 主线程之外，还存在一个”任务队列”（task queue）。系统把异步任务放到”任务队列”之中，然后继续执行后续的任务。
- 一旦”执行栈”中的所有任务执行完毕，系统就会读取”任务队列”。如果这个时候，异步任务已经结束了等待状态，就会从”任务队列”进入执行栈，恢复执行。
- 主线程不断重复上面的第三步。

事件循环(event loops) browsing contexts 和 web workers
任务队列可以有多个, 两类

- 任务队列 任务源

- setTimeout macrotask queue (宏任务)
- promise microtask queue (微任务) Object.observe  MutationObserver

- 相同任务源的任务，只能放到一个任务队列中。
- 不同任务源的任务，可以放到不同任务队列中。
- （同一个任务队列，能否容纳不同任务源的任务，没说）

### 事件循环完整版

1.在tasks队列中选择最老的一个task,用户代理可以选择任何task队列，如果没有可选的任务，则跳到下边的microtasks步骤。
2.将上边选择的task设置为正在运行的task。
3.Run: 运行被选择的task。
4.将event loop的currently running task变为null。
5.从task队列里移除前边运行的task。
6.Microtasks: 执行microtasks任务检查点。（也就是执行microtasks队列里的任务）
7.更新渲染（Update the rendering）...
9.返回到第一步。

### microtasks检查点

当用户代理去执行一个microtask checkpoint，如果microtask checkpoint的flag（标识）为false，用户代理必须运行下面的步骤：
1.将microtask checkpoint的flag设为true。
2.Microtask queue handling: 如果event loop的microtask队列为空，直接跳到第八步（Done）。
3.在microtask队列中选择最老的一个任务。
4.将上一步选择的任务设为event loop的currently running task。
5.运行选择的任务。
6.将event loop的currently running task变为null。
7.将前面运行的microtask从microtask队列中删除，然后返回到第二步（Microtask queue handling）。
8.Done: 每一个environment settings object它们的 responsible event loop就是当前的event loop，会给environment settings object发一个 rejected promises 的通知。
9.清理IndexedDB的事务。
10.将microtask checkpoint的flag设为flase。

````javascript
setTimeout(function(){console.log(4)},0);
new Promise(function(resolve){
console.log(1)
for( var i=0 ; i<10000 ; i++ ){
i==9999 && resolve()
}
console.log(2)
}).then(function(){
console.log(5)
});
console.log(3);
// 12354
````

````javascript
setTimeout(function(){
console.log(4);
new Promise(function(resolve){
for( var i=0 ; i<10000 ; i++ ){
i==9999 && resolve()
}
}).then(function(){
console.log(6)
});
},0);

setTimeout(function(){console.log(7);},0);

new Promise(function(resolve){
console.log(1)
for( var i=0 ; i<10000 ; i++ ){
i==9999 && resolve()
}
console.log(2)
}).then(function(){
console.log(5)
});
console.log(3);
// 1235467
````

## 数据存储

### 跨域

一个域名的 JS ，在未经允许的情况下，不得读取另一个域名的内容。

## 总结

- [ ] DTD(Document Type Definition)
- [ ] 全量(增量)布局(绘制)   动态变化
- [ ] CSS解析
- [ ] 词法分析和语法分析
- 事件循环

- https://github.com/aooy/blog/issues/5
- https://www.zhihu.com/question/36972010

