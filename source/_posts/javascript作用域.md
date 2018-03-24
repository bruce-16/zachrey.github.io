---
title: "javascript作用域"
date: 2017-02-11 13:39:23
tags: "scope"
categories: "javascript"
---

>作用域是一套规则，用于确定在何处以及如何查找变量（标识符）。作用域的使用提高了程序逻辑的局部性，增强程序的可靠性，减少名字冲突。

## 词法作用域
考虑以下代码：
{% codeblock lang:javascript %}
    function foo(a) {
	   var b = a * 2;
	   function bar(c) {
	      console.log( a, b, c );
	   }
	   bar( b * 3 );
	}
	foo( 2 ); // 2, 4, 12
{% endcodeblock %}
在这个例子中有三个逐级嵌套的作用域。为了帮助理解，可以将它们想象成几个逐级包含
的气泡。
{% asset_img 1.png 气泡 %}

<!-- more -->

1. 包含着整个全局作用域，其中只有一个标识符： foo 。
2. 包含着 foo 所创建的作用域，其中有三个标识符： a 、 - bar 和 b 。
3. 包含着 bar 所创建的作用域，其中只有一个标识符： c 。

>词法作用域意味着作用域是由书写代码时函数声明的位置来决定的。
>
>JavaScript 中有两个机制可以“欺骗”词法作用域： eval(..) 和 with 。但一般不建议使用，这里不多做介绍。

## 函数作用域和块作用域
JavaScript 具有基于函数的作用域，意味着每声明
一个函数都会为其自身创建一个气泡，而其他结构都不会创建作用域气泡。但事实上这并不完全正确，下面我们来看一下。

考虑下面的代码：
{% codeblock lang:javascript %}
function foo(a) {
    var b = 2;
    // 一些代码
    function bar() {
    // ...
    }
    // 更多的代码
    var c = 3;
}
{% endcodeblock %}

在这个代码片段中， foo(..) 的作用域气泡中包含了标识符 a 、b 、 c 和 bar 。无论标识符声明出现在作用域中的何处，这个标识符所代表的变量或函数都将附属于所处作用域的气泡。
bar(..) 拥有自己的作用域气泡。全局作用域也有自己的作用域气泡，它只包含了一个标识符： foo 。
由于标识符 a 、 b 、 c 和 bar 都附属于 foo(..) 的作用域气泡，因此无法从 foo(..) 的外部对它们进行访问。也就是说，这些标识符全都无法从全局作用域中进行访问，因此下面的代码会导致 ReferenceError 错误：
　　bar(); // 失败
　　console.log( a, b, c ); // 三个全都失败
但是，这些标识符（ a 、 b 、 c 、 foo 和 bar ）在 foo(..) 的内部都是可以被访问的，同样在bar(..) 内部也可以被访问（假设 bar(..) 内部没有同名的标识符声明）。

### 隐藏内部实现
　　对函数的传统认知就是先声明一个函数，然后再向里面添加代码。但反过来想也可以带来一些启示：从所写的代码中挑选出一个任意的片段，然后用函数声明对它进行包装，实际上就是把这些代码“隐藏”起来了。
　　实际的结果就是在这个代码片段的周围创建了一个作用域气泡，也就是说这段代码中的任何声明（变量或函数）都将绑定在这个新创建的包装函数的作用域中，而不是先前所在的作用域中。换句话说，可以把变量和函数包裹在一个函数的作用域中，然后用这个作用域来“隐藏”它们。
例如：
{% codeblock lang:javascript %}
function doSomething(a) {
    b = a + doSomethingElse( a * 2 );
	console.log( b * 3 );
}
function doSomethingElse(a) {
	return a - 1;
}
var b;
doSomething( 2 ); // 15
{% endcodeblock %}
在这个代码片段中，变量 b 和函数 doSomethingElse(..) 应该是 doSomething(..) 内部具体实现的“私有”内容。给予外部作用域对 b 和 doSomethingElse(..) 的“访问权限”不仅没有必要，而且可能是“危险”的，因为它们可能被有意或无意地以非预期的方式使用，从而导致超出了  doSomething(..) 的适用条件。更“合理”的设计会将这些私有的具体内容隐藏在 doSomething(..) 内部，例如：
{% codeblock lang:javascript %}
function doSomething(a) {
    function doSomethingElse(a) {
        return a - 1;
    }
    var b;
    b = a + doSomethingElse( a * 2 );
    console.log( b * 3 );
}
doSomething( 2 ); // 15
{% endcodeblock %}
现在， b 和 doSomethingElse(..) 都无法从外部被访问，而只能被 doSomething(..) 所控制。功能性和最终效果都没有受影响，但是设计上将具体内容私有化了，设计良好的软件都会依此进行实现。
### 函数作用域
我们已经知道，在任意代码片段外部添加包装函数，可以将内部的变量和函数定义“隐藏”起来，外部作用域无法访问包装函数内部的任何内容。
{% codeblock lang:javascript %}
var a = 2;
function foo() { // <-- 添加这一行
  var a = 3;
  console.log( a ); // 3
} // <-- 以及这一行
foo(); // <-- 以及这一行
console.log( a ); // 2
{% endcodeblock %}
　　虽然这种技术可以解决一些问题，但是它并不理想，因为会导致一些额外的问题。首先，必须声明一个具名函数 foo() ，意味着 foo 这个名称本身“污染”了所在作用域（在这个例子中是全局作用域）。其次，必须显式地通过函数名（ foo() ）调用这个函数才能运行其中的代码。如果函数不需要函数名（或者至少函数名可以不污染所在作用域），并且能够自动运行，这将会更加理想。
　　幸好，JavaScript 提供了能够同时解决这两个问题的方案:
{% codeblock lang:javascript %}
var a = 2;
( function foo(){ // <-- 添加这一行
    var a = 3;
    console.log( a ); // 3
})(); // <-- 以及这一行
console.log( a ); // 2
{% endcodeblock %}
(function foo(){ .. }) 作为函数表达式意味着 foo 只能在 .. 所代表的位置中被访问，外部作用域则不行。 foo 变量名被隐藏在自身中意味着不会非必要地污染外部作用域。
### 匿名和具名
对于函数表达式最熟悉的场景可能就是回调参数了，比如：
{% codeblock lang:javascript %}
setTimeout( function() {
    console.log("I waited 1 second!");
}, 1000 );
{% endcodeblock %}
>这叫作匿名函数表达式，因为 function().. 没有名称标识符。函数表达式可以是匿名的，
而函数声明则不可以省略函数名——在 JavaScript 的语法中这是非法的。

匿名函数表达式书写起来简单快捷，很多库和工具也倾向鼓励使用这种风格的代码。但是它也有几个缺点需要考虑。

1. 匿名函数在栈追踪中不会显示出有意义的函数名，使得调试很困难。
2. 如果没有函数名，当函数需要引用自身时只能使用已经过期的 arguments.callee 引用，
比如在递归中。另一个函数需要引用自身的例子，是在事件触发后事件监听器需要解绑
自身。
3. 匿名函数省略了对于代码可读性 / 可理解性很重要的函数名。一个描述性的名称可以让
代码不言自明。

行内函数表达式非常强大且有用——匿名和具名之间的区别并不会对这点有任何影响。给函数表达式指定一个函数名可以有效解决以上问题。始终给函数表达式命名是一个最佳实践：
{% codeblock lang:javascript %}
setTimeout( function timeoutHandler() { // <-- 快看，我有名字了！
    console.log( "I waited 1 second!" );
}, 1000 );
{% endcodeblock %}
### 立即执行函数表达式
{% codeblock lang:javascript %}
var a = 2;
(function foo() {
    var a = 3;
    console.log( a ); // 3
})();
console.log( a ); // 2
{% endcodeblock %}
　　　由于函数被包含在一对 ( ) 括号内部，因此成为了一个表达式，通过在末尾加上另外一个( ) 可以立即执行这个函数，比如 (function foo(){ .. })() 。第一个 ( ) 将函数变成表达式，第二个 ( ) 执行了这个函数。
　　　这种模式很常见，几年前社区给它规定了一个术语：IIFE，代表立即执行函数表达式（Immediately Invoked 　Function Expression）；
　　　函数名对 IIFE 当然不是必须的，IIFE 最常见的用法是使用一个匿名函数表达式。虽然使用具名函数的 IIFE 并不常见，但它具有上述匿名函数表达式的所有优势，因此也是一个值得推广的实践。
　　　相较于传统的 IIFE 形式，很多人都更喜欢另一个改进的形式： (function(){ .. }()) 。仔细观察其中的区别。第一种形式中函数表达式被包含在 ( ) 中，然后在后面用另一个 () 括号来调用。第二种形式中用来调用的 () 括号被移进了用来包装的 ( ) 括号中。
　　　这两种形式在功能上是一致的。选择哪个全凭个人喜好。
　　　IIFE 的另一个非常普遍的进阶用法是把它们当作函数调用并传递参数进去。
例如：
{% codeblock lang:javascript %}
var a = 2;
(function IIFE( global ) {
var a = 3;
console.log( a ); // 3
console.log( global.a ); // 2
})( window );
console.log( a ); // 2
{% endcodeblock %}
　　我们将 window 对象的引用传递进去，但将参数命名为 global ，因此在代码风格上对全局
　　对象的引用变得比引用一个没有“全局”字样的变量更加清晰。当然可以从外部作用域传递任何你需要的东西，并将变量命名为任何你觉得合适的名字。这对于改进代码风格是非常有帮助的。
### 块作用域
>除 JavaScript 外的很多编程语言都支持块作用域，因此其他语言的开发者对于相关的思维方式会很熟悉，但是对于主要使用 JavaScript 的开发者来说，这个概念会很陌生。

{% codeblock lang:javascript %}
for (var i=0; i<10; i++) {
    console.log( i );
}
{% endcodeblock %}
我们在 for 循环的头部直接定义了变量 i ，通常是因为只想在 for 循环内部的上下文中用 i ，而忽略了 i 会被绑定在外部作用域（函数或全局）中的事实。
这就是块作用域的用处。变量的声明应该距离使用的地方越近越好，并最大限度地本地化。另外一个例子：
{% codeblock lang:javascript %}
var foo = true;
if (foo) {
    var bar = foo * 2;
    bar = something( bar );
    console.log( bar );
}
{% endcodeblock %}
bar 变量仅在 if 声明的上下文中使用，因此如果能将它声明在 if 块内部中会是一个很有意义的事情。但是，当使用 var 声明变量时，它写在哪里都是一样的，因为它们最终都会属于外部作用域。
####  try/catch
非常少有人会注意到 JavaScript 的 ES3 规范中规定 try / catch 的 catch 分句会创建一个块作用域，其中声明的变量仅在 catch 内部有效。
{% codeblock lang:javascript %}
try {
    undefined(); // 执行一个非法操作来强制制造一个异常
}
catch (err) {
    console.log( err ); // 能够正常执行！
}
console.log( err ); // ReferenceError: err not found
{% endcodeblock %}
####  let
到目前为止，我们知道 JavaScript 在暴露块作用域的功能中有一些奇怪的行为。如果仅仅是这样，那么 JavaScript 开发者多年来也就不会将块作用域当作非常有用的机制来使用了。
幸好，ES6 改变了现状，引入了新的 let 关键字，提供了除 var 以外的另一种变量声明方式。
let 关键字可以将变量绑定到所在的任意作用域中（通常是 { .. } 内部）。换句话说， let为其声明的变量隐式地了所在的块作用域。
{% codeblock lang:javascript %}
var foo = true;
if (foo) {
    let bar = foo * 2;
    bar = something( bar );
    console.log( bar );
}
console.log( bar ); // ReferenceError
{% endcodeblock %}
 **let 循环:**
一个 let 可以发挥优势的典型例子就是之前讨论的 for 循环。
{% codeblock lang:javascript %}
for (let i=0; i<10; i++) {
    console.log( i );
}
console.log( i ); // ReferenceError
{% endcodeblock %}
下面通过另一种方式来说明每次迭代时进行重新绑定的行为：
{% codeblock lang:javascript %}
{
    let j;
    for (j=0; j<10; j++) {
        let i = j; // 每个迭代重新绑定！
        console.log( i );
    }
}
{% endcodeblock %}
####  const
除了 let 以外，ES6 还引入了 const ，同样可以用来创建块作用域变量，但其值是固定的（常量）。之后任何试图修改值的操作都会引起错误。
{% codeblock lang:javascript %}
var foo = true;
if (foo) {
     var a = 2;
     const b = 3; // 包含在 if 中的块作用域常量
     a = 3; // 正常 !
     b = 4; // 错误 !
}
console.log( a ); // 3
console.log( b ); // ReferenceError!
{% endcodeblock %}
### 小结
　　函数是 JavaScript 中最常见的作用域单元。本质上，声明在一个函数内部的变量或函数会在所处的作用域中“隐藏”起来，这是有意为之的良好软件的设计原则。
　　但函数不是唯一的作用域单元。块作用域指的是变量和函数不仅可以属于所处的作用域，也可以属于某个代码块（通常指 { .. } 内部）。
　　从 ES3 开始， try/catch 结构在 catch 分句中具有块作用域。
　　在 ES6 中引入了 let 关键字（ var 关键字的表亲），用来在任意代码块中声明变量。 if(..) { let a = 2; } 会声明一个劫持了 if 的 { .. } 块的变量，并且将变量添加到这个块中。
　　有些人认为块作用域不应该完全作为函数作用域的替代方案。两种功能应该同时存在，开发者可以并且也应该根据需要选择使用何种作用域，创造可读、可维护的优良代码。
## 提升
>到现在为止，已经很熟悉作用域的概念，以及根据声明的位置和方式将变量分配给作用域的相关原理了。函数作用域和块作用域的行为是一样的，可以总结为：任何声明在某个作用域内的变量，都将附属于这个作用域。
但是作用域同其中的变量声明出现的位置有某种微妙的联系。

### 先有鸡还是先有蛋
直觉上会认为 JavaScript 代码在执行时是由上到下一行一行执行的。但实际上这并不完全正确，有一种特殊情况会导致这个假设是错误的。
考虑以下代码：
{% codeblock lang:javascript %}
a = 2;
var a;
console.log( a );
{% endcodeblock %}
你认为 console.log(..) 声明会输出什么呢？
很多人会认为是 **undefined** ，因为 var a 声明在 a = 2 之后，他们自然而然地认为变量被重新赋值了，因此会被赋予默认值 undefined 。但是，真正的输出结果是 **2** 。

### 编译器
当你看到 var a = 2; 时，可能会认为这是一个声明。但 JavaScript 实际上会将其看成两个声明： var a; 和 a =2; 。第一个定义声明是在编译阶段进行的。第二个赋值声明会被留在原地等待执行阶段。
我们的第一个代码片段会以如下形式进行处理：
{% codeblock lang:javascript %}
var a;
a = 2;
console.log( a );
{% endcodeblock %}
因此，打个比方，这个过程就好像变量和函数声明从它们在代码中出现的位置被“移动”到了最上面。这个过程就叫作提升。
看着这样的代码：
{% codeblock lang:javascript %}
foo();
function foo() {
    console.log( a ); // undefined
    var a = 2;
}
{% endcodeblock %}
foo 函数的声明（这个例子还包括实际函数的隐含值）被提升了，因此第一行中的调用可以正常执行。
变成这样：
{% codeblock lang:javascript %}
function foo() {
    var a;
    console.log( a ); // undefined
    a = 2;
}
foo();
{% endcodeblock %}
可以看到，函数声明会被提升，但是函数表达式却不会被提升。
{% codeblock lang:javascript %}
foo(); // 不是 ReferenceError, 而是 TypeError!
var foo = function bar() {
// ...
};
{% endcodeblock %}
### 函数优先
函数声明和变量声明都会被提升。但是一个值得注意的细节（这个细节可以出现在有多个“重复”声明的代码中）是函数会首先被提升，然后才是变量。
{% codeblock lang:javascript %}
foo(); // 1
var foo;
    function foo() {
    console.log( 1 );
}
foo = function() {
    console.log( 2 );
};
{% endcodeblock %}
会输出 1 而不是 2 ！这个代码片段会被引擎理解为如下形式：
{% codeblock lang:javascript %}
function foo() {
    console.log( 1 );
}
foo(); // 1
foo = function() {
    console.log( 2 );
};
{% endcodeblock %}
注意， var foo 尽管出现在 function foo()... 的声明之前，但它是重复的声明（因此被**忽略**了），因为函数声明会被提升到普通变量之前。
尽管重复的 var 声明会被忽略掉，但出现在后面的函数声明还是可以覆盖前面的。
{% codeblock lang:javascript %}
foo(); // 3
function foo() {
    console.log( 1 );
}
var foo = function() {
    console.log( 2 );
};
function foo() {
    console.log( 3 );
}
{% endcodeblock %}
### 小结
　　我们习惯将 var a = 2; 看作一个声明，而实际上 JavaScript 引擎并不这么认为。它将 var a和 a = 2 当作两个单独的声明，第一个是编译阶段的任务，而第二个则是执行阶段的任务。
　　这意味着无论作用域中的声明出现在什么地方，都将在代码本身被执行前首先进行处理。
　　可以将这个过程形象地想象成所有的声明（变量和函数）都会被“移动”到各自作用域的最顶端，这个过程被称为提升。

>代码段来至于**《你不知道的javascript》**