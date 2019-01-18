---
title: Hexo Mathjax下标的问题
date: 2016-10-20 17:38:47
categories: 网站
tags: Hexo
toc:
mathjax: true
---
不得不说Hexo虽然优点不少，但是毕竟年轻，问题也比较多。在Latex里下标是用`_`来表示下标，但是在Hexo里`_`会被渲染成`<em>`标签，导致数学公式里出现多个下标被当成加粗，如`$x_1 x_2$`会变成$x*1 x*2$，不是我们想要的，所以要怎么办？

---
在网上找了找，发现一个不是很好的解决方法，就是把Hexo对`_`的渲染删掉，找到博客根目录下的，
`_node_modules/hexo-renderer-marked/node_modules/marked/lib/marked.js`，
在`em`的正则匹配里删掉关于`_`的内容，如下，

{% codeblock %}
em: /^\b_((?:[^_]|__)+?)_\b|^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
{% endcodeblock %}
改成
{% codeblock %}
em: /^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
{% endcodeblock %}
还有一处，
{% codeblock %}
em: /^_(?=\S)([\s\S]*?\S)_(?!_)|^\*(?=\S)([\s\S]*?\S)\*(?!\*)/
{% endcodeblock %}
改成
{% codeblock %}
em: /^\*(?=\S)([\s\S]*?\S)\*(?!\*)/
{% endcodeblock %}
然后就解决了，不过这方法有明显的缺点，其一就是以后加粗只能用`*`了，而且这个插件更新的话，可能还要再改一次，由于对Hexo代码了解少，目前也只能这样了。

---
参考链接：
1. <http://kubicode.me/2016/03/16/Hexo/Fix-Hexo-Bug-In-Mathjax/>
