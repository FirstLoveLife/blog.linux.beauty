---
title: mathjax的初次尝试
date: 2017-01-10 09:45:01
categories: [others]
thumbnail: http://pledgie.com/assets/campaigns/27873/medium/MJ-logo.gif?1420119498
bgimage: http://pledgie.com/assets/campaigns/27873/medium/MJ-logo.gif?1420119498
author: "Li Chen"
---
### 缘起

*昨天花了1个多小时学习了Markdown这个轻量级编辑语言Markdown 语法说明(简体中文版)的语法后，动了将数学笔记也用wiz的内置markdown编辑器来电子化的念想，毕竟我是个非常崇尚脱离纸质环境的人，谁叫我是环境工程专业的呢:disappointed:。于是又日常地碰到了一大堆需要[谷歌][1]来解决的问题，* **烦在其中，乐在其中**。
### 工具介绍
* [$mathjax$][2]
* [$Editor.md插件$][3]
* [$latex$][4]

### 过程
1. 遇到的第一个问题是 $\Delta$这个符号如何打出，没想到它也恰恰是所有问题中最后解决的。
2. 第一反应自然是[Google一下][5],结果出来的教程都是 `
\Delta `这个操作
3. 尝试百遍后都无一例外的失败了，然后开始了折腾之旅
1. 可能是wiz自带的markdown对Latex支持不完善，于是调用了[Org2Wiz][6]这个插件（此插件需要mathjax和emacs的支持）,又反复使用编辑了org，mdp等格式，依然无解，
2. 接下来开始怀疑是电脑中罗马字符的数字字库，于是谷歌一番，发现的确有提及 
> 下面的表格中将给出在数学模式中常用的所有符号。使用表 3.12–3.167
所列出的符号，必须事先安装 AMS 数学字库并且在文档的导言区加载宏
包： amssymb。如果你的系统中没有安装 AMS 宏包和数学字库，可去下述
地址下载：
CTAN:/tex-archive/macros/latex/required/amslatex

3. 之后的安装过程中涉及到了 $MiKTeX$，因为安装宏包必须用它。
4. 然而配置好了一切后，依然无法给我一个 $\Delta$，看着预览中的\Delta，百撕不得骑姐。
3. 无奈之下只好重新开始通读Latex的教程[一份其实很短的 LaTeX 入门文档 | 始终](wiz://open_document?guid=9e50bd28-95e2-4113-b13f-b226f178cb0f&kbguid=26a35e8e-c84b-4670-bdf1-48aeaff9aa00),发现了**解决的方法**
> LaTeX 的数学模式有两种：行内模式(inline)和行间模式(display)。前者在正文的行文中，插入数学公式；后者独立排列单独成行。
在行文中，使用`$ ... $`可以插入行内公式，使用`\[ ... \]`可以插入行间公式，如果需要对行间公式进行编号，可以使用`equation`环境：


可见，单独的\delta并不能直接实现$\delta$这个罗马字符的print，必须在两端都加上`$`才行。

### 总结
虽然看起来除了最后一步，之前的一切都是白折腾了，大一的我可能认为就是这样的，毕竟对解决这一问题来说，这些就是**无用功**，但是谁知道以后的路上会不会就遇到了这些细枝末节的details呢？我坚信，一定会的。

[1]:  https://www.google.com/search?q=markdown%E5%A6%82%E4%BD%95%E5%BC%95%E7%94%A8%E6%9C%AC%E8%BA%AB%E8%AF%AD%E6%B3%95&ie=&oe=#q=%E4%B8%BA%E7%9F%A5%E7%AC%94%E8%AE%B0+latex  "谷歌"
[2]: https://en.wikipedia.org/wiki/MathJax  "mathjax"
[3]: https://en.wikipedia.org/wiki/MathJax "Editor.md插件"
[4]: https://www.latex-project.org/ "latex"
[5]:https://www.google.com/search?q=delta+latex&ie=&oe= "google一下"
[6]: https://www.google.com/search?q=delta+latex&ie=&oe= "Org2Wiz"

