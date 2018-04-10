---
title: 为什么React组件点击事件回调函数会需要绑定this
date: 2018-04-10 13:36:14
categories: React
tags:
- React
- ES6
- class
- this
---
<blockquote class="blockquote-center">
世界上没有比快乐更能使人美丽的化妆品。——布雷顿
</blockquote>

js里面的this绑定是代码执行的时候进行绑定的，而不是编写的时候，所以this的指向取决于函数调用时的各种条件。

关于this的几种绑定规则和分析可以参考《你不知道的javascript》上卷对this的详解。

## 这里简单的说明一下this的**默认绑定**：
```javascript
var a = "foo";
function foo(){
  console.log(this.a);
}
foo();  // "foo"
```
<!-- more -->
在浏览器环境中，单独运行的函数，也就是没有跟任何对象产生关系的情况下执行该函数，此时的this会指向`window`对象，在Node环境中会指向`global`对象。所以上面代码中this指向了window，然后调用了找到了window中的变量a。

当使用严格模式后：
```javascript
var a = "foo";
function foo(){
  "use strict";
  console.log(this.a);
}
foo();  // TypeError：this is undefined
```

这里关于默认绑定的另一种情况需要注意：***在严格模式下，全局对象将无法使用默认绑定，因此this会绑定到undefined。在ES6的class中，也会绑定到undefined***

再看代码：
```javascript
class A {
  constructor(){
    this.aa = 1;
  }
  hello(){
    console.log(this.aa);
  }

  hello2() {
   (function(){
      console.log(this.aa);
   })();
  }

}

const a = new A();
a.hello(); // 1
a.hello2(); // Uncaught TypeError: Cannot read property 'aa' of undefined
```
证明了上面的绑定规则，绑定到了undefined。

## react组件
有了上面的铺垫，现在来谈谈react中的组件。在ES6中，react创建组件也就是使用class创建了一个“类”。
```javascript
class View extends React.Component {
    constructor(props){
        super(props);
        this.state={
            
        }
    }
    
    handleClick(e) {
        console.log(e);
    }
    
    render(){
        return (
            <a onClick={this.handleClick}>click me</a>
        );
    }
}
```

主要将注意力放在JSX的语法中，其中点击的回调方法对函数进行了this的绑定。但是前面`a.hello();`不是可以正常输出么？正常绑定了this么？为什么这里还需要进行绑定？

## JSX
想要搞清楚为什么需要绑定this，就需要搞清楚JSX到底是一个什么东西。我们看react官方的描述:

本质上来讲，JSX 只是为 `React.createElement(component, props, ...children)` 方法提供的语法糖。比如下面的代码：
```
<MyButton color="blue" shadowSize={2}>
  Click Me
</MyButton>
```
编译为：
```
React.createElement(
  MyButton,
  {color: 'blue', shadowSize: 2},
  'Click Me'
)
```
如果没有子代，你还可以使用自闭合标签，比如：

```
 <div className="sidebar" />
```
编译为：
```
React.createElement(
  'div',
  {className: 'sidebar'},
  null
)
```
> [react官网深入jsx（中文）](https://doc.react-china.org/docs/jsx-in-depth.html)

## 去JSX后的react组件
根据官网的描述，上面写的 `View`组件就变成了如下形式
```javascript
class View extends React.Component {
    constructor(props){
        super(props);
    }
    
    handleClick(e) {
        console.log(e);
    }
    
    render(){
        return React.createElement(
            "a",
            { onClick: this.handleClick},
            "click me"
        );
    }
}
```
### 分析
我们看`React.createElement`的第二个参数，**传入的是一个对象**，而这个对象里面有属性的值是取`this`** 对象里面的属性 **，当这个对象放入`React.createElement`执行后，去取这个`this.handleClick`属性时候，this已经不是我们在书写的时候认为的绑定在`View`上了。`this.handleClick`这里的this就会像`a.hello2`的里面`this`绑定一样，`this`会默认绑定，但是又是在ES6的`class`中，所以`this`绑定了`undefined`，说到这就能说明标题了"为什么React组件点击事件回调函数会需要绑定this？"。

所以需要手动绑定this，在react中：(或者箭头函数)
```javascript
class View extends React.Component {
    constructor(props){
        super(props);
    }
    
    handleClick(e) {
        console.log(e);
    }
    
    render(){
        return (
            <a onClick={this.handleClick.bind(this)}>click me</a>
        );
    }
}
```

> 有错误或者描述不清晰的地方，欢迎指正~
