---
title: 使用Hexo+github创建个人博客
tags: technology
categories: Hexo
date: 2017-01-12 21:54:26
---


## 准备工作
* nodejs的安装
* github账号
* 域名

最先，还是以官方的文档为最权威的说明：[官方文档](https://hexo.io/zh-cn/docs/)

首先，我们需要先保证电脑上安装了nodejs和git，过程中是需要用到git bash。

* [git](https://www.git-scm.com/)
* [nodejs](https://nodejs.org/en/)

<!-- more -->

## 开始
>现在是默认电脑已经安装好了以两个东西。

先自己在电脑中新建一个文件夹，至于哪个盘没有限制，高兴就好。不过还是建议能将文件归档分类，这是一个好习惯，以后找起来方便。

{% asset_img folder.png . %}

进入文件夹，在空白的位置右击鼠标，选择Git Bash Here，就会出现git的命令行工具，是一个小黑框。如果git没有安装好，那么右键菜单上面是没有这个选项的。那么现在我们在这个命令行工具上输入下面的命令：
``` bash
$ npm install -g hexo-cli
```
nodejs安装完成后，就会自带的安装npm，这个npm是nodejs包管理工具，这里就是用npm安装hexo客户端， -g意思是全局安装，以后随便哪个文件下都可以使用hexo。

接下来就是使用hexo来生产博客需要的文件。
``` bash
$ hexo init
```
顺序执行之后，刚才的文件夹（我这里是Blog）里面多了很多文件。
不要急，接下来继续执行一下命令
``` bash
$ hexo g
$ hexo s
```
出现的这样画面：

{% asset_img hexos.png . %}

浏览器中出现的画面：

{% asset_img 3.png . %}

这样，本地的博客是能正常运行了。

## Git的设置

创建一个Repository
{% asset_img 4.png . %}
建立与你用户关联的仓库，仓库的名字必须要为“your_name.github.io”,这是一个固定写法，这个写法只是在这里是特殊的，正常使用github仓库名字没必要这样。

{% asset_img 5.png . %}

接下来就需要修改hexo的配置文件，来将本地与github仓库远程关联上来。下图是目录结构，配置文件就在博客根目录（我这里是Blog文件夹）。名字为： _config.yml

{% asset_img 6.png . %}

在文件中搜索：deploy，将它设置为：(注意冒号后面有一个空格)
{% codeblock %}
    deploy:
           type: git
           repo: https://github.com/your_name/your_name.github.io
           branch: master
{% endcodeblock %}
    
{% asset_img 7.png . %}

然后安装部署工具：
``` bash
$ npm install hexo-deployer-git --save
```
安装成功后，执行:
``` bash
$ hexo deploy
```
或者：
``` bash
$ hexo d
```

这时候hexo g命令生成在public文件夹下面的静态文件就会部署到github上你刚创建的仓库里面。

在浏览器上输入： “your_name.github.io”就会出现最开始在本地开启的博客页面。如果出现DNS未找到，或者网页错误，那么先等待片刻，过3~5分钟再试试。如果还出现错误，请仔细检查上面每一步操作，检查下nodejs，git的安装和配置，特别是git的设置。关于git的设置请参考:[Git安装与配置 - 铁锚的CSDN博客 - 博客频道 - CSDN.NET](http://blog.csdn.net/renfufei/article/details/41647875)


至此，如果我们每次发布新日志，一般的步骤为：
{% codeblock %}
    hexo clean

    hexo generate

    hexo deploy
{% endcodeblock %}
## 域名解析
下面以万网购买的域名为例。我是通过阿里云上的万网购买的域名，进入到阿里云的网页管理界面。
先进入到万网的管理页面，找到域名解析栏目：

{% asset_img 8.png . %}

{% asset_img 9.png . %}

这里添加两个解析：（记录类型）CNAME， （主机记录）www和@
，（记录值）your_name.github.io

很重要的一步：
　　在博客目录下的source文件夹内创建文件“CNAME”，名字一定不要写错，并且不能有后缀名，文件里面的内容为，你的域名。注意，不要书写www。
{% asset_img 10.png . %}
现在执行：
``` bash
$ hexo g 
$ hexo deploy
```
　　将新文件重新生成，然后再部署到github上。到这里，就可以尝试使用你的域名去访问新建立的博客了。域名解析时间一般在10分钟以内，如果访问出现404，就先等待5分钟左右，但一般都会立即生效。
>如果还是404就仔细检查CNAME的内容，并跑到你的github仓库中看看有没有CNAME文件，并且里面的内容是否正确。因为hexo deploy如果没有正常执行的话，CNAME文件是部署不到github仓库上去的。这里就再三声明，如果github上没有，那就再检查你的git配置。
{% asset_img 11.png . %}

## 最后

给大家推荐个主题
[Next](http://theme-next.iissnan.com/)  Elegant Theme for Hexo

* [文档](http://theme-next.iissnan.com/getting-started.html)   这里有详细的使用说明。


{% asset_img 12.png . %}


>具体博客的操作，更改请参考[Hexo官网文档](https://hexo.io/zh-cn/docs/)