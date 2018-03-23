---
title: "JavaScript同步、异步、回调执行顺序之经典闭包setTimeout分析"
date: 2017-04-09 11:45:52
tags: technology
categories: javascript
---

> [https://segmentfault.com/a/1190000008922457](https://segmentfault.com/a/1190000008922457 "原文出处")

同步、异步、回调？傻傻分不清楚。

大家注意了，教大家一道口诀：

**同步优先、异步靠边、回调垫底（读起来不顺）**

用公式表达就是：

**同步 => 异步 => 回调**

这口诀有什么用呢？用来对付面试的。

有一道经典的面试题：

{% codeblock lang:javascript %}
for (var i = 0; i < 5; i++) {
    setTimeout(function() {
        console.log('i: ',i);
    }, 1000);
}

console.log(i);
{% endcodeblock %}

{% codeblock lang:javascript %}
//输出
5
i:  5
i:  5
i:  5
i:  5
i:  5
console.log(i);
{% endcodeblock %}

这道题目大家都遇到过了吧，那么为什么会输出这个呢？记住我们的口诀 同步 => 异步 => 回调

1、for循环和循环体外部的console是同步的，所以先执行for循环，再执行外部的console.log。（同步优先）

2、for循环里面有一个setTimeout回调，他是垫底的存在，只能最后执行。（回调垫底）

那么，为什么我们最先输出的是5呢？

非常好理解，for循环先执行，但是不会给setTimeout传参（回调垫底），等for循环执行完，就会给setTimeout传参，而外部的console打印出5是因为for循环执行完成了。

这里涉及到JavaScript执行栈和消息队列的概念，概念的详细解释可以看阮老师的 JavaScript 运行机制详解:再谈Event Loop - 阮一峰的网络日志，或者看 并发模型与Event Loop

{% asset_img 1.png . %}

《图片来自于MDN官方》

我拿这个例子做一下讲解，JavaScript单线程如何处理回调呢？JavaScript同步的代码是在堆栈中顺序执行的，而setTimeout回调会先放到消息队列，for循环每执行一次，就会放一个setTimeout到消息队列排队等候，当同步的代码执行完了，再去调用消息队列的回调方法。

在这个经典例子中，也就是说，先执行for循环，按顺序放了5个setTimeout回调到消息队列，然后for循环结束，下面还有一个同步的console，执行完console之后，堆栈中已经没有同步的代码了，就去消息队列找，发现找到了5个setTimeout，注意setTimeout是有顺序的。

那么，setTimeout既然在最后才执行，那么他输出的i又是什么呢？答案就是5。。有人说不是废话吗？

现在告诉大家为什么setTimeout全都是5，JavaScript在把setTimeout放到消息队列的过程中，循环的i是不会及时保存进去的，相当于你写了一个异步的方法，但是ajax的结果还没返回，只能等到返回之后才能传参到异步函数中。在这里也是一样，for循环结束之后，因为i是用var定义的，所以var是全局变量（这里没有函数，如果有就是函数内部的变量），这个时候的i是5，从外部的console输出结果就可以知道。那么当执行setTimeout的时候，由于全局变量的i已经是5了，所以传入setTimeout中的每个参数都是5。很多人都会以为setTimeout里面的i是for循环过程中的i，这种理解是不对的。

===========================================分割线=========================================

看了上面的解释，你是不是有点头晕，没事，继续深入讲解。

我们给第一个例子加一行代码。

{% codeblock lang:javascript %}
for (var i = 0; i < 5; ++i) {
    setTimeout(function() {
        console.log('2: ',i);
    }, 1000);
    console.log('1: ', i); //新加一行代码
}

console.log(i);

{% endcodeblock %}


{% codeblock lang:javascript %}
//输出
1:  0
1:  1
1:  2
1:  3
1:  4
5
2:  5
2:  5
2:  5
2:  5
2:  5
{% endcodeblock %}

来，大家再跟着我一起念一遍：同步 => 异步 => 回调 （强化记忆）

这个例子可以很清楚的看到先执行for循环，for循环里面的console是同步的，所以先输出，for循环结束后，执行外部的console输出5，最后再执行setTimeout回调 55555。。。

=====================================分割线============================================

这么简单，不够带劲是不是，那么面试官会问，怎么解决这个问题？

最简单的当然是let语法啦。。

{% codeblock lang:javascript %}
for (let i = 0; i < 5; ++i) {
    setTimeout(function() {
        console.log('2: ',i);
    }, 1000);
}

console.log(i);

{% endcodeblock %}

{% codeblock lang:javascript %}
//输出
i is not defined
2:  0
2:  1
2:  2
2:  3
2:  4
{% endcodeblock %}

咦，有同学问，为什么外部的i报错了呢？又有同学问，你这个口诀在这里好像不适应啊？

let是ES6语法，ES5中的变量作用域是函数，而let语法的作用域是当前块，在这里就是for循环体。在这里，let本质上就是形成了一个闭包。也就是下面这种写法一样的意思。如果面试官对你说用下面的这种方式，还有let的方式，你可以严肃的告诉他：这就是一个意思！这也就是为什么有人说let是语法糖。

{% codeblock lang:javascript %}
var loop = function (_i) {
    setTimeout(function() {
        console.log('2：', _i);
    }, 1000);
};

for (var _i = 0; _i < 5; _i++) {
    loop(_i);
}

console.log(i);
{% endcodeblock %}

面试官总说闭包、闭包、闭包，什么是闭包？后面再讲。

写成ES5的形式，你是不是发现就适合我说的口诀了？而用let的时候，你发现看不懂？那是因为你没有真正了解ES6的语法原理。

我们来分析一下，用了let作为变量i的定义之后，for循环每执行一次，都会先给setTimeout传参，准确的说是给loop传参，loop形成了一个闭包，这样就执行了5个loop，每个loop传的参数分别是0，1，2，3，4，然后loop里面的setTimeout会进入消息队列排队等候。当外部的console执行完毕，因为for循环里的i变成了一个新的变量 _i ，所以在外部的console.log(i)是不存在的。

现在可以解释闭包的概念了：当内部函数以某一种方式被任何一个外部函数作用域访问时，一个闭包就产生了。

我知道你又要我解释这句话了，loop(_i)是外部函数，setTimeout是内部函数，当setTimeout被loop的变量访问的时候，就形成了一个闭包。（别说你又晕了）

随便举个新的例子。
{% codeblock lang:javascript %}
function t() {
    var a = 10;
    var b = function() {
        console.log(a);    
    }
    b();
}
t(); //输出 10
{% endcodeblock %}

跟我一起念口诀：同步 => 异步 => 回调 （强化记忆）
先执行函数t，然后js就进入了t内部，定义了一个变量，然后执行函数b，进入b内部，然后打印a，这里都是同步的代码，没什么异议，那么这里怎么解释闭包：函数t是外部函数，函数b是内部函数，当函数b被函数t的变量访问的时候，就形成了闭包。

========================================分割线==============================================

上面主要讲了同步和回调执行顺序的问题，接着我就举一个包含同步、异步、回调的例子。

{% codeblock lang:javascript %}
let a = new Promise(
  function(resolve, reject) {
    console.log(1)
    setTimeout(() => console.log(2), 0)
    console.log(3)
    console.log(4)
    resolve(true)
  }
)
a.then(v => {
  console.log(8)
})

let b = new Promise(
  function() {
    console.log(5)
    setTimeout(() => console.log(6), 0)
  }
)

console.log(7)
{% endcodeblock %}

看到这个例子，千万不要害怕😨，先读一遍口诀：同步 => 异步 => 回调 （强化记忆）

1、看同步代码：a变量是一个Promise，我们知道Promise是异步的，是指他的then()和catch()方法，Promise本身还是同步的，所以这里先执行a变量内部的Promise同步代码。（同步优先）

{% codeblock lang:javascript %}
console.log(1)
    setTimeout(() => console.log(2), 0) //回调
    console.log(3)
    console.log(4)
{% endcodeblock %}

2、Promise内部有4个console，第二个是一个setTimeout回调（回调垫底）。所以这里先输出1，3，4回调的方法丢到消息队列中排队等着。

3、接着执行resolve(true)，进入then()，then是异步，下面还有同步没执行完呢，所以then也滚去消息队列排队等候。（真可怜）（异步靠边）4、b变量也是一个Promise，和a一样，执行内部的同步代码，输出5，setTimeout滚去消息队列排队等候。

5、最下面同步输出7。

6、同步的代码执行完了，JavaScript就跑去消息队列呼叫异步的代码：异步，出来执行了。这里只有一个异步then，所以输出8。

7、异步也over，轮到回调的孩子们：回调，出来执行了。这里有2个回调在排队，他们的时间都设置为0，所以不受时间影响，只跟排队先后顺序有关。则先输出a里面的回调2，最后输出b里面的回调6。

8、最终输出结果就是：1、3、4、5、7、8、2、6。

我们还可以稍微做一点修改，把a里面Promise的 setTimeout(() => console.log(2), 0)改成 setTimeout(() => console.log(2), 2)，对，时间改成了2ms，为什么不改成1试试呢？1ms的话，浏览器都还没有反应过来呢。你改成大于或等于2的数字就能看到2个setTimeout的输出顺序发生了变化。所以回调函数正常情况下是在消息队列顺序执行的，但是使用setTimeout的时候，还需要注意时间的大小也会改变它的顺序。

====================================分割线==================================================

口诀不一定是万能的，只能作为一个辅助，更重要的还是要理解JavaScript的运行机制，才能对代码执行顺序有清晰的路线。

还有async/await等其他异步的方案，不管是哪种异步，基本都适用这个口诀，对于新手来说，可以快速读懂面试官出的js笔试题目。以后再也不用害怕做笔试题啦。

特殊情况下不适应口诀的也很正常，JavaScript博大精深，不是一句话就能概括出来的。

最后，在跟着我念一遍口诀：同步 => 异步 => 回调