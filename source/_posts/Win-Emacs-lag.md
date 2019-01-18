---
title: 解决Windows下Emacs Unicode字符爆卡的问题
date: 2017-06-08 04:48:39
categories: Windows
tags: Emacs
toc:
mathjax:
---
### 发现问题

博主在Windows下用Emacs编辑Latex的时候遇到严重卡顿，情况是在出现数学符号时，Prettify-Symbols（一个minor mode）会自动把数学符号转化成Unicode字符显示出来，比如`\theta`会显示成`Θ`，这本来时很赞的功能，但是在Windows下出现这种符号就卡，卡到按一下键盘都要反应半天。这个问题困扰了博主很长时间，一直找不到原因。在网上搜索的时候，看到[一个帖子](https://emacs-china.org/t/windows10-emacs/992)，里面说Emacs出现中文就非常卡，解决方法是换成Windows已安装的字体，这给了楼主启发，因为中文也是Unicode字符，所以博主意识到这很有可能是字体问题。

### 解决

事实证明博主是对的，把字体换了确实就没问题了。[rolandwalker@github](https://github.com/rolandwalker/unicode-fonts)搞了一个包来配置Unicode字体，直接用起来就好了，在Emacs执行`M-x`，输入`package-install`回车，输入`unicode-fonts`安装，完成后在`~/.emacs`（如果你有其他配置，也可能在其他文件）加入

```
;; Setup Unicode fonts
(require 'unicode-fonts)
(unicode-fonts-setup)
```

### 手动配置

装好上面的东西之后基本能用了，但是最好手动设置一下，因为上面的很多字体不存在，建议安装[Dejavu字体](http://dejavu-fonts.org)（Sans和Sans Mono）和Google的[Noto字体](https://www.google.com/get/noto/)（Sans, Sans Mono和Sans Symbols），然后在Emacs执行
`M-x` `customize-group` `RET` `unicode-fonts` `RET`

然后可以配置一下自己想要的字体，并且将`Unicode Fonts Tweaks`->`Unicode Fonts Existence Checks`改成`Only First Existing Font for Each Block`，这样的话会自动剔除系统没有的字体。


最后重启Emacs，试一下是不是如丝般顺滑？