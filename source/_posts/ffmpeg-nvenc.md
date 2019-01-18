---
title: ffmpeg用GPU转码
date: 2016-11-08 23:36:29
categories: Windows
tags: Tools
toc: true
mathjax:
---
有些视频编码太屌，播放起来对CPU消耗巨大，而在电视上或者电视盒子上看视频，编码就需要合适，楼主用的FireTV Stick，用起KODI确实吃力，放些编码太强的视频是不行的，所以就想到了先在电脑上转码，其实转码最专业的莫过于[ffmpeg](https://www.ffmpeg.org/)，但是因为是命令行工具，一开始偷懒不想用，于是试了[Handbrake](https://handbrake.fr/)，也是开源，不过也只是把ffmpeg包转一下而已，转一个105分钟的视频为mp4竟然用了一个多小时，而且楼主的电脑配置还是挺不错，i5-6600K 16G DDR4 3200MHZ GTX970 +500G ssd. 这样依然用了一个多小时，真是在逗我。主要原因还是CPU利用率太低了，于是想到既然是视频相关，为什么不能用GPU呢，毕竟论能力GPU还是比CPU强大，于是发现确实有这种东西，却是很新的技术，最后还是逃不过ffmpeg.

### 平台
GPU转码，自然要分是谁的GPU，包括
1. NVDIA的[NVENC](https://developer.nvidia.com/nvidia-video-codec-sdk)，之前叫CUDA.
2. AMD的[VEC](http://developer.amd.com/community/blog/2014/02/19/introducing-video-coding-engine-vce/)
3. Intel的[QSV](http://www.intel.com/content/www/us/en/architecture-and-technology/quick-sync-video/quick-sync-video-general.html)

不过ffmpeg貌似只支持NVENC和QSV，AMD的还没有支持。博主的是GTX970便NVENC为例。

### 前提

1. 驱动版本要求，`Linux: >=367.35，Windows: >=368.69`. 还是非常新的。
2. ffmpeg支持，编译时不要`--disable-nvenc`

### 编译ffmpeg
ffmpeg官网是不提供编译好的windows版的，而是由[zeranoe](https://ffmpeg.zeranoe.com/builds/)编译的，博主未试。大家可以试试。也可以用[cygwin](https://www.cygwin.com/)，不过楼主还是自己编译了。

毕竟是开源的东西，在windows编译比较麻烦，就在linux上cross compile，已经有人做了一键编译脚本，在[github](https://github.com/rdp/ffmpeg-windows-build-helpers)上。直接放在linux运行就行了，编译需要硬盘空间10G.

### 转换

编译完成后拷贝到windows，并放到自己的PATH，可以在CMD或者PowerShell运行。具体的参数嘛，可是有些麻烦，因为你先要了解视频编码的各种知识，具体看官方的[手册](https://www.ffmpeg.org/ffmpeg.html)。
mp4编码必是主流，也是好用，这是我用的参数，自行调整
{% codeblock %}
ffmpeg -i input_video -c:v h264_nvenc -profile:v high -level 4.1 -preset fast -b:v 7M -pix_fmt yuv420p output.mp4
{% endcodeblock %}

上面的参数适合1080P视频，效果颇是不错，还可以用`-r`限制fps，比如`-r 24`。`-c:v h264_nvenc`是用NVENC进行编码，`-pix_fmt yuv420p` 对第十代（GTX10xx）之前都是必须的，和颜色编码相关，因为ffmpeg默认使用`yuv444p`，然而这只在第十代之后支持，所以不限定就会报错。
转换105分钟视频只用了大概18分钟，相比之前的一个多小时真是太快了。

### 添加字幕
如果想在转换过程中添加字幕，若输出格式为`mp4`，可以添加`-f ass(or srt) -i subtitle_file -c:s mov_text`，若输出格式为`mkv`，可以添加`-i subtitle_file -c:s copy`，如果想给一个视频只加字幕(`mp4`)，可以使用：
{% codeblock %}
ffmpeg -i input_video -f ass(or srt) -i subtitle_file -c:v copy -c:a copy -c:s mov_text output.mp4
{% endcodeblock %}

