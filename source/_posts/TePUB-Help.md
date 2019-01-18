---
title: TePUB使用教程
date: 2017-07-25 19:45:45
categories: TePUB
tags: [TePUB, Tools] 
toc: true
mathjax:
---

## 简介

[TePUB](https://github.com/shengdie/TePUB-bin/releases)可以将`txt`格式的书转成精排的`epub`、`mobi`（包括`KF8`和`mobi7`），适用于`windows desktop 10.14393  x86 x64`及以上。~~可以在[Windows商店](https://www.microsoft.com/store/apps/9nhvm5jzv01w)下载~~。目前被下架了，因为之前网站挂掉了，说隐私声明打不开，重新上架很麻烦，不想弄了，就在[github](https://github.com/shengdie/TePUB-bin/releases)发布离线安装包了。下载解压，里面有说明。

## txt文件要求

本软件对`txt`小说源文件有一定要求，如下：
- 编码是`Unicode`(`UTF-8`等)或者`GBK`，推荐`Unicode`
- 章节标题单独占一行，如果章节标题后没有换行，便紧跟章节内容，是无法正确处理的。如

{% codeblock 正确结构 %}
第一章 xxx 
（章节内容）
{% endcodeblock %}

{% codeblock 错误结构 %}
第一章 xxx （章节内容）
{% endcodeblock %}

- 一行即一段。
- <font color="blue">自动识别</font>多级目录如“部-卷-章”，要求高级标题和低级标题之间没有内容，如

```
第1卷 xxx    #卷标题和章标题之间不能有内容（可以有空行）
第一章 xxx
（章节内容）
第二章 xxx
（章节内容）
……
第2卷
第一章 xxx
……
```
<font color="blue">如果有内容，如“卷首语”，可以使用自定义多级目录（目前仅支持两级）。</font>

## 图文步骤
### 基本功能
![基本功能](http://i.imgur.com/1sb8qKL.png)
对于多级目录，注意主规则的正则表达式要能匹配每一级的标题，如
```
第一卷 xxx
第一章 xxx
```

主规则可以是`^\s*第[一二三四五六七八九十零〇百千两]+[章卷]`

```
卷一 xxx
第一卷 xxx
```
主规则可以是`^\s*(第[一二三四五六七八九十零〇百千两]+章|卷[一二三四五六七八九十零〇两]+)`

总之主规则要能匹配所以级别的标题。不懂正则表达式的可以看[正则表达式30分钟入门](https://deerchao.net/tutorials/regex/regex.htm)。

---

### 自定义多级目录（1.1.29新增功能）

如果你的文件，高级目录和低级目录之间有内容，如每卷之前都有“卷首语”，那么你可以使用自定义功能，如下
![自定义多级目录](http://i.imgur.com/YnTUfjb.png)
目前自定义仅支持两级，需要输入一级和二级目录的正则表达式匹配规则，如图中所示。多级目录规则将取代主规则。

---

### 章节调整

选好文件，选好主规则，额外规则，点击章节调整，就会看到
![章节调整](http://i.imgur.com/o1p1YN5.png)
注意
- 主章节总数只算最深级目录。
- 章节预览只是简单内容预览，并没有应用CSS
- 章节内容编辑，是转成`HTML`之后的结果，不要删除`<p> </p>`

---

### 重命名章节

有的文件，在章节命名上编号比较混乱，比如有的是阿拉伯数字，有的是中文数字，这个时候可以用重命名章节，把它统一一下，如下
![重命名章节](http://i.imgur.com/LHEMQ1k.png)
数字前后可以自定义，并且可以替换标题中的奇怪字符为空格。

---

### 书籍信息

![书籍信息](http://i.imgur.com/js1Xsci.png)
可以输入书籍的各种信息，全部可选。

---

### 输出样式

![输出样式](http://i.imgur.com/7pmcEfm.png)

可以选择内置字体，可以自定义CSS，里面可以调整页边距等。

---

### 输出格式

![输出格式](http://i.imgur.com/ZK9cZAd.png)

生成`epub`是最快的，`mobi`的话要调用`kindlegen`，速度比较慢，注意`KF8`实际上是`AZW3`，但是后缀若改成`.azw3`，放到kindle上无法显示封面，所以后缀依然用`mobi`，但是后缀改成`mobi`又不能在`Kindle Previewer 3`里预览，所以你要预览就把后缀改成`.azw3`。