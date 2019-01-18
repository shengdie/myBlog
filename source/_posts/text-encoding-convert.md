---
title: 文本文件编码转换
date: 2016-11-03 19:38:13
categories: Linux
tags: ShellCommands
toc: ture
mathjax:
---
在Wingdows里，中文编码一般是`GBK`，而在Linux一般采取`UTF-8`，所以在Linux打开中文文本的时候经常是乱码，所以要进行编码转换。
### 文件内容编码转换
有一傻瓜式终端工具`enca`可以轻易转码。
此工具需要另外安装，使用非常简单。
{% codeblock %}
enca -L LANGUAGE files
{% endcodeblock %}

此命令用于查看文件编码，其中`LANGUAGE`指定文本语言，`files`则是想要测试的文件。比如
{% codeblock %}
enca -L zh_CN test.txt
{% endcodeblock %}
想要转换编码则是，
{% codeblock %}
enca -L zh_CN -x UTF-8 test.txt
{% endcodeblock %}

则可文件编码转为`UTF-8`。

### 文件名编码转换
有时后一些中文名字的文件在Linux下名称会乱码，这类文件一般在Widows下命名，编码多为`GBK`，所以在Linux乱码，而`convmv`则可以进行文件名编码转换。
{% codeblock %}
convmv -f original_encoding -t target_encoding files
{% endcodeblock %}

这个命令并不能真正重命名，只会显示转换之后的效果，如果发现确实没有差错，可以`--notest`进行真正重命名。
另外`convmv`还有一个功能就是，将网址里的编码转换回来，比如将`%20`转换成空格，只需添加`--unescape`即可。



