---
title: 关于自由度和限制
date: 2016-10-19 22:51:49
categories: Physics
tags: 
toc:
mathjax: true
---
今天看一个Oxford U的一个关于spinor的讲义，里面谈到转动群$ SO(3) $时有一句话让我感觉非常奇怪，说：
{% blockquote %}
The matrix group $SO(3)$ is three dimensional because a general $3\times 3$ matrix has nine parameters, but the orthogonality and unit determinant conditions together set six constraints.
{% endblockquote %}

这句话的表述是有问题的，其实正交本身便给出了6个限制，而unit determinant只是我们的选择或者定义附加的条件罢了。
首先我们要明确一下什么是自由度，一个自由度应该是指有一个变量在定义域内可以自由变换，什么是限制呢？给一个限制便少了一个自由度。比如考察两个实变量$x,y \in \mathcal{R}$, 在不加任何限制的青款下就有两个自由度，如果加一个方程如$ x+y = 1 $，也就是一个限制，便少了一个自由度，自由度便是$1$.

然后回到我们最初的问题，对一个三维转动矩阵$R$，满足$RR^T=1$，这个条件便给出6个方程，也就是6个限制，那$det(R)=1$是什么呢？实际上，如果正交已经满足，那么$det(RR^T)=1 \Rightarrow det(R)^2=1 \Rightarrow det(R)= \pm 1$，所以才说$det(R)=1$是定义的一个附加条件，它并不给出一个自由度的限制，比如我们把$det(R)=1$的转动群叫特殊转动群，甚至我们不需要加$det(R)=1$，只需要加个$det(R)>0$就行了。这个问题实际类似解方程$x^2=1 \Rightarrow x= \pm 1$，而我们加个附加条件$x>0$一样。

