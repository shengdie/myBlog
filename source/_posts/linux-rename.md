---
title: Linux 批量重命名带空格的文件
date: 2016-11-02 02:55:31
categories: Linux
tags: ShellCommands
toc:
mathjax:
---
专家请忽略本文。Linux/Unix的文件是不该用特殊字符命名的，因为处理比较困难。博主现在遇到了问题，文件夹内有几百个文件名带空格的文件，想要重命名，按照以往的批处理方式不成功，因为空格前后被当成了独立的字符串。怎么处理呢？
{% codeblock %}
for f in *\ *; do mv "$f" "${f/%.lrc/ - .lrc}"; done
{% endcodeblock %}

这个命令会把所有名为`*.lrc`的文件命名为`* - .lrc`文件。
如果想将所有空格换成`_`则是
{% codeblock %}
for f in *\ *; do mv "$f" "${f// /_}"; done
{% endcodeblock %}

其实linux有另外一个很强大的`rename`命令，批量命名文件非常好，支持正则匹配，不过博主目前多正则表达式还一窍不通，不能给出什么例子。但是`rename`自有处理文件名带特殊字符的方法。比如
{% codeblock %}
rename --nows *
{% endcodeblock %}

会将当前目录所有文件名中的空格替换成`_`。以后等博主研究清楚再更新吧。