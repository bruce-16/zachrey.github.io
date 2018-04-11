---
title: JS中NAN和Infinity两个特殊值需要注意的地方
date: 2018-04-11 11:01:45
tags:
- JS
- 类型
- js-number
categroies: javascript
---

<blockquote class="blockquote-center">
每一天都以许下希望开始，以收获经验结束。
</blockquote>

现在来讨论讨论在JS中操作数值进行计算的时候，会出现的两个特殊值：**NaN** 和 **Infinity**。

# NaN
在JS中，它被解释为"not a number",一般在将一个字符串解析成数字时候出现的错误：
```javascript
Number('XYZ'); // NaN
```

NaN与所有的值都不相等，哪怕就是算与自己相比较都不相等。
```javascript
NaN === NaN; // false
NaN === 1;   // false
...
```
<!-- more -->

**但是请注意**使用**typeof**计算NaN的类型的时候，它竟然返回的是`number`。。。
```javascript
typeof NaN;   // 'number'
```
很是无语。

## 怎么判断一个值是否为NaN？
NaN是javascript唯一个和自身不相等的值，所以我们不能使用"=="或者"==="来判断它，我们需要使用ES6提供`Number.isNaN`来判断它，ES6之前有一个全局函数`isNaN`也可以用来判断。
```javascript
Number.isNaN(NaN); // true
```
`Number.isNaN`和`isNaN`这两个函数的表现形式是不相同的：
```javascript
isNaN('xyz'); // true
Number.isNaN('xyz'); // false
```
从这里看出`isNaN`会将参数强制转换为数值类型，然后再去判断，但是`Number.isNaN`不会这么做。

## Number的强制转换
当使用关键字`Number`进行类型显示转换的时候，里面究竟做了什么呢？这里简单说明下。

看如下代码：

```javascript
let obj = {
    valueOf: function (){ return 2; }
}
Number(obj); // 2

obj = {
    valueOf: function (){ return NaN; }
}
const objNum = Number(obj); // NaN
Number.isNaN(obj);       // false
Number.isNaN(objNum);    // true
isNaN(obj);              // true
```
这里可以看出，当使用`Number`强转的时候，会去调用值原型上的方法`valueOf`，当我们重写这个方法后，就可以转换为指定的值，上面也清晰的解释了`Number.isNaN`和`isNaN`的区别。


# Infinity
当你去除以一个`0`时，就会得到这个特殊值。
```javascript
3 / 0          // Infinity
Infinity / 0   // Infinity
```
**但是请注意：**
```
NaN / 0        // NaN
```
`NaN`这个值与其他值进行操作还是为`NaN`。

对于`Infinity`你不能再与一个`Infinity`进行相互作用，不然就会得到`NaN`。
```javascript
Infinity / Infinity // NaN
Infinity - Infinity // NaN
```
但是可以做乘和加的运算
```javascript
Infinity + Infinity // Infinity
Infinity * Infinity // Infinity
```
因为这个值表象为`无穷`,上面四个运算就是符合数学逻辑的。


> 有错误或者描述不清晰的地方，欢迎指正~

!['干巴爹'](/uploads/ganbadie.jpg)
