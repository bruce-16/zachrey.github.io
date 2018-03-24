---
title: javascript正则表达式
tags: regexp
categories: javascript
date: 2017-01-18 09:59:00
---

# javascript & 正则表达式
>正则表达式，又称规则表达式。（英语：Regular Expression，在代码中常简写为regex、regexp或RE），计算机科学的一个概念。正则表通常被用来检索、替换那些符合某个模式(规则)的文本。
>正则表达式并不是javascript所特有的，很多语言都支持正则，但是每种语言对正则表达式的支持或多或少会存在差别。

<!-- more -->

### 正则表达式在js中的存在形式
在js中，我们可以使用RegExp和字面量来创建正则表达式的对象。
RegExp对象：
{% codeblock lang:javascript %}
var re = new RegExp('abc');
{% endcodeblock %}
字面量：
{% codeblock lang:javascript %}
var re = /abc/;
{% endcodeblock %}
正则表达式有三种模式：**（全局）global**， **（忽略大小写）ignoreCase**， **（多行搜索）multiLines**
开启的方式为：
{% codeblock lang:javascript %}
var re = new RegExp('abc','igm');
{% endcodeblock %}
或者
{% codeblock lang:javascript %}
var re = /abc/igm;
{% endcodeblock %}
- i	-> 执行对大小写不敏感的匹配。
- g	-> 执行全局匹配（查找所有匹配而非在找到第一个匹配后停止）。
- m	-> 执行多行匹配。


### 正则表达式组成形式
- **原义字符** ： 字符本身的意思 ，例如 a就是字母a
- **元字符**  ：在正则表达式中有特殊含义的非字母字符，例如：* + ^ $ . | \ ( ) { } [ ] 

>/a/ /a*/ /a+/ ...

### 字符类
一般情况下正则表达式里面的一个字符对应字符串中的一个字符。
>/abc/ 对应字符串"abc"，不对应"abcd"和"ab"等。

使用"[ ]"元字符来构建一个类（类是一个泛指符合某些特征性对象，而不是特殊的指一个字符）。
>/[abc]/ 指的可以匹配***一个***字符，这个字符必须是a,b,c三个中的其中一个。
>/[**^**abc]/ 可以匹配***一个***字符，这个字符必须不是a,b,c三个中的其中任何一个。

{% codeblock lang:javascript %}
    str = 'a1b2c3d4';

    result = str.replace(/[abc]/g, 'X');  //匹配a,b,c三个中任何一个。  a or b or c
    console.log(result); //X1X2X3d4

    result = str.replace(/[^abc]/g, 'X'); //匹配不为a,b,c三个中任何一种。 !a and !b and !c
    console.log(result); //aXbXcXXX
{% endcodeblock %}

### 范围类
> [a-z] 可以匹配***一个***字符，改字符必须为a,b,c,d...z 中的一个
> [a-zA-Z0-9] 范围可以连写

### 预定义类
预定义类     | 等价            | 含义
------|-----------------|----
.    | [^\r\n]         | 除了回车和换行符之外的字符
\d   | [0-9]           | 数字字符
\D   | [^0-9]          | 非数字字符
\s   | [\t\n\x0B\f\r]  | 空白符
\S   | [^\t\n\x0B\f\r] | 非空白字符
\w   | [a-zA-Z0-9]     | 单词字符（字母，数字，下划线）
\W   | [^a-zA-Z0-9]    | 非单词字符
{% codeblock lang:javascript %}
    str = 'ab1/';

    result = str.replace(/ab\d./, 'X');   // ab + 数字 + 除了\n的任意字符
    console.log(result); // X
{% endcodeblock %}

### 边界
预定义类     | 含义            | 例子
-----|-----------------|----
^    | 以XXXX开始      | ^a
$    | 以XXXX结束      | a$
\b   | 单词边界        | \ba\b
\B   | 非单词边界      | \Ba\B
{% codeblock lang:javascript %}
    str = 'This is a dog.';

    result = str.replace(/is/g, '0');   //只要是is，就会被替换
    console.log(result); //Th0 0 a dog.

    result = str.replace(/\bis\b/g, '0');   //只有is为一个单独的单词，才会被替换
    console.log(result); //This 0 a dog.

    result = str.replace(/\Bis\b/g, '0');   //只有is前面不为单词开始，后面是单词结束。才会被替换
    console.log(result); //Th0 is a dog.

    str = '@123@456@789@';

    result = str.replace(/@./g, '0');    //替换“@+除了\n的任意字符”为“0”
    console.log(result); //023056089@

    result = str.replace(/^@./g, '0');  //替换“@+除了\n的任意字符”并且是字符串开头为“0”
    console.log(result); //023@456@789@

    result = str.replace(/.@$/g, '0');    //替换“除了\n的任意字符+@”并且是字符串结尾为“0”
    console.log(result); //@123@456@780
{% endcodeblock %}

### 量词
预定义类     | 含义    
-----|-----------------
?    | 出现0次或者1次  
+    | 出现1次或多次      
*    | 出现0次或多次        
{n}  | 出现n次 
{n,m}| 出现n次到m次
{n,} | 至少出现n次
{0,m}| js不支持{,m},这里表示出现0次到m次
{% codeblock lang:javascript %}
    str = '123456789';

    result = str.replace(/\d*/, '0'); //将所有的数字都替换成0
    console.log(result); //0
{% endcodeblock %}

### 贪婪模式和非贪婪模式
在js中，正则表达式默认的就是贪婪模式，使用非贪婪模式就需要在量词的后面加上一个“?”。
> /\d*?/ /\d{1,5}?/ /\d+?/ /\d??/ ... 

{% codeblock lang:javascript %}
    str = '123456789';
    
    result = str.replace(/\d*/, '0');
    console.log(result); //0

    result = str.replace(/\d{3,6}/, '0');  //贪婪模式，默认就是
    console.log(result); // 0789

    result = str.replace(/\d{3,6}?/, '0');  //非贪婪模式
    console.log(result); // 0456789
{% endcodeblock %}
贪婪模式会尽可能的去匹配所有字符，非贪婪会尽可能少的来匹配。如上代码。

### 分组和或运算
使用"( )"可以达到分组的功能。如下，使量词作用与分组。
> (zachrey){3} 表明 zachrey应该出现三次
> zachrey(3)  表明 字符串最后的“y”应该出现三次

使用“|”可以达到或运算的效果
> zachrey | zhang  => 1.zachreyhang  2.zachrezhang
> (zachrey) | (zhang) => 1.zachrey  2.zhang

### 反向引用
现有一个要求： 将时间 years-months-days 格式的时间装换为 months/days/years 格式，例如"2017-01-22" 装换为 "01/22/2017",时间内容可能变化，但是格式固定。
那么有这样的要求的时候，就需要使用反向引用来解决。在js正则表达式中有这样的字符样式 $1, $2, $3, $4...它们分别表示的是分组1、分组2、分组3、分组4...里面的内容。
前提情况下，正则表达式里面有分组，没有的话，它们就会变为原义字符。
{% codeblock lang:javascript %}
    str = "2017-01-22";

    result = str.replace(/(\d{4})-(\d{2})-(\d{2})/, "$2/$3/$1"); // $2 第二个分组
    console.log(result); //01/22/2017
{% endcodeblock %}

##### 忽略分组
如果不希望分组被$捕获，只需要在分组里面加上“?:”。
> /(?:zachrey)(zhang)/     $1 =>>>  zhang

### 前瞻
正则表达式从文本头部向尾部开始解析，那么现在的尾部被称为“前”，开始的头部称为“后”。
前瞻就是在正则表达式匹配到规则的时候向前检查是否符合断言。后顾和前瞻的方向相反，js并不支持后顾。
符合和不符合特定的断言称为肯定/正向匹配和否定/负向匹配。
> 正向前瞻： exp(?=assert)
> 负向前瞻:  exp(?!assert)

{% codeblock lang:javascript %}
    str = 'a1b*c4';

    result = str.replace(/[a-z](?=\d)/g, 'X'); //替换字母并且该字母后面跟的是数字
    console.log(result); //X1b*X4

    result = str.replace(/[a-z](?!\d)/g, 'X'); //替换字母并且该字母后面跟的不是数字
    console.log(result); //a1X*c4
{% endcodeblock %}

### 正则表达式对象的属性
* global : 是否全文搜索，默认false
* ignore case : 是否大小写敏感，默认false
* multiline : 多行搜索模式，默认false
* lastindex : 是当前表达式匹配内容的最后一个字符的下一个字符的位置下标
* source : 正则表达式的文本字符串

> 这些属性都是只读属性

### 常用方法
* [Regexp.prototype.test()](http://www.w3school.com.cn/jsref/jsref_test_regexp.asp)
* [Regexp.prototype.exec()](http://www.w3school.com.cn/jsref/jsref_exec_regexp.asp)
* [String.prototype.search()](http://www.w3school.com.cn/jsref/jsref_search.asp)
* [String.prototype.match()](http://www.w3school.com.cn/jsref/jsref_match.asp)
* [String.prototype.replace()](http://www.w3school.com.cn/jsref/jsref_replace.asp)
* [String.prototype.split()](http://www.w3school.com.cn/jsref/jsref_split.asp)

