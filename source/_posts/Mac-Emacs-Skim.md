---
title: Mac下配置Emacs+AucTex+Skim
date: 2017-05-29 23:45:22
categories: Mac
tags: [Emacs, Latex]
toc: true
mathjax:
---

### 目标

不得不说，Emacs确实很强大，就是配置比较麻烦，本文给出Emacs+AucTex+Skim配置推荐，个人觉得比较好用。用Emacs编辑Latex，用Skim预览，用Skim的原因是为了实现反向搜索，就是从pdf可以跳回tex对应的位置。

### 准备

#### 安装和配置Emacs

推荐使用[Homebrew](https://brew.sh/)安装，

{% codeblock %}
brew install emacs --with-cocoa --with-gnutls
brew linkapps emacs #可以从软件列表打开
{% endcodeblock %}

安装完成后，推荐使用[purcell@github](https://github.com/purcell/emacs.d)提供配置，用起来确实很舒服。点进去之后下面有介绍使用方法，实际只要clone一些就好了。

打开Emacs，等它自动获取各种插件，建议配置一下主题，`M-x customize-themes`,选一个自己喜欢的主题。

#### 安装AucTex和Skim

`M-x package-list-packages`, 搜索到AucTex，安装就可以了。或者可以`M-x package-install`，输入`auctex`。

Skim是开源的专门为Mac搞的pdf阅读器，非常强大，去[官网](http://skim-app.sourceforge.net/)下载安装即可。

### 配置AucTex和Skim

#### Auctex
在`~/.emacs.d/lisp/init-locales.el`里加入一下代码，
{% codeblock %}
(setq TeX-PDF-mode t)

(setq TeX-view-program-list
      '(("PDF Viewer" "/Applications/Skim.app/Contents/SharedSupport/displayline -b -g %n %o %b")))

(eval-after-load 'tex
  '(progn
     (assq-delete-all 'output-pdf TeX-view-program-selection)
     (add-to-list 'TeX-view-program-selection '(output-pdf "PDF Viewer"))))

(add-hook 'LaTeX-mode-hook
          #'(lambda ()
              (add-to-list 'TeX-command-list '("pdfLaTeX" "%`pdflatex -synctex=1%(mode)%' %t" TeX-run-TeX nil t))
              (setq TeX-command-extra-options "-file-line-error -shell-escape")
              (setq TeX-command-default "pdfLaTeX")
              (setq TeX-save-query  nil ) ;; 不需要保存即可编译
              ))

(custom-set-variables
 '(TeX-source-correlate-method 'synctex)
 '(TeX-source-correlate-mode t)
 '(TeX-source-correlate-start-server t))
{% endcodeblock %}

#### Skim

打开，`Skim -> Preferences -> Sync -> PDF-Tex Sync support`
```
Preset: Custom
Command: /usr/local/bin/emacsclient
Arguments: --no-wait +%line "%file"
```

自此配置完成，在Emacs `C-c C-v`可以正向搜索，在Skim `Shift-Command + Click`可以反向搜索。


