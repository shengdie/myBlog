---
title: Hexo自定义404页面
date: 2016-10-13 01:46:43
categories: 网站
tags: Hexo
toc: ture
---

### 写一个404.html

首先你要写一个`404.html`，怎么写我就不多说了，看自己的想法。但要注意一点，就是Hexo默认会将所有的html用主题渲染，导致的后果就是，404页面会被搞成一个Post，我写这篇文章也主要是因为我遇到了这个坑。网上搜了很久，甚至想了解一下Hexo主题怎么写，浪费了不少时间，最后再[这里](https://github.com/hexojs/hexo/issues/158 "git")发现了解决方式。
按上面所说要在html最前面添加：

{% codeblock %}
layout: false
---
{% endcodeblock %}
但是貌似有没有有效还要看主题里有没有判断，这个不同的主题就不同了，博主的主题貌似已经加了判断，所以只要加了上面的代码就OK了，不过目测大部分主题都是支持的。

最后把写好的`404.html`放在博客主目录下的`source`文件夹再运行`hexo g`和`hexo d`就行了。效果可以参考博主的[404页面](http://qingtian.ml/404.html "404")。

### 修改Nginx配置

如果你用了`Nginx`，你只写404网页也是无效的，不存在的网址打开依然是Nginx默认的错误页，想生效你还要稍微配置下Nginx：
在Nginx配置文件`/etc/nginx/nginx.conf`的`http`区加入
{% codeblock %}
fastcgi_intercept_errors on;
{% endcodeblock %}
在自己网站的配置文件（一般在`/etc/nginx/conf.d/`）的`server`区加入（你也可以在`nginx.conf`加入）
{% codeblock %}
error_page 404  /404.html;
{% endcodeblock %}
最后`service nginx restart`重启Nginx.

### 公益404

如果你有心，现在有一些公益的404项目，比如[腾讯公益404](http://www.qq.com/404/ "腾讯404")，[益云404](http://yibo.iyiyun.com/Home/Index/web404 "益云404")，想加入只要在`404.html`里加入他们提供的代码就可以了。博主本来也是想加腾讯404的，但是发现腾讯提供的返回主页代码实效了，变成了返回腾讯网，不知道是不是故意的，所以我感觉腾讯目的不纯，不是真的为了公益，就没有加。