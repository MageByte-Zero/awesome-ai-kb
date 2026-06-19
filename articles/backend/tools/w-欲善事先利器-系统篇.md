# 欲善事先利器——系统篇

> 工欲善其事，必先利其器，好鞋踢好球是非常合乎逻辑的事情。
>
> ——《长江七号》

我们的目标是提高编程技术能力。或是面向兴趣编程(FOM, favorite oriented programming)，或是面向钱途编程(MOM, money oritented programming)，抑或真的是面向”对象“编程(SOM, spouse oriented programming)，所有这些，都需要你提高自己的技术能力，才能如火纯青，游刃有余。

那么今天，我却不讲如何提高技术能力。

我讲什么？讲效率。工欲善其事必先利其器。今天不藏私，将我珍藏多年的百宝箱一一推荐给大家。这里面都是一些小工具，可以提高我们平时编码和工作的效率。有其则事半功倍矣。

> 使孤成大业者，必此人也。 —— 曹操

![哈哈](https://magebyte.oss-cn-shenzhen.aliyuncs.com/common/caocao.png)

## Chocolatey

链接：https://chocolatey.org/

如何像 Linux 一样在 windows 下安装软件？

试想一下，每次重装系统，都要安装一堆常用的小软件，不胜其烦。这个**win 下的包管理工具**，可以帮助到你。你可能听过 Mac 的 Homebrew，deb 的 apt-get，centos 的 yum。Chocolatey 就是 win 上的 Homebrew。你可以通过一条命令来安装大部分软件。也有人调侃，win 上我们也应该像程序员一样安装软件！

下面是我用的一些软件。喜欢的可以选择安装。有些我也会在此文中推荐。

```bash

rem 安装chocolatey
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"

choco install -y everything
choco install -y listary
choco install -y googlechrome
choco install -y wox
choco install -y autohotkey.portable
choco install -y bandizip

choco install -y jdk8

choco install -y git.install
choco install -y maven

choco install -y intellijidea-community
choco install -y intellijidea-ultimate
choco install -y vscode
choco install -y notepad++
choco install -y vim

choco install -y cmder
choco install -y cmdermini

choco install -y python2
choco install -y vcpython27
choco install -y python3
choco install -y pycharm

choco install -y fiddler
choco install -y jmeter

choco install -y sqlyog
choco install -y postman
```

## Everything

链接：https://www.voidtools.com/zh-cn/

在 windows 上快速搜索文件和目录的软件。

这款软件是我常用的，win 自带的搜索确实不好用。**everything 软件小，搜索速度快，支持通配符查找**。我经常需要查找文件，但从不在资源管理器里面点来点去，只要我对文件名有点印象，都直接在 everything 中搜索；有时候需要打开一个路径很深的文件(比如：hosts)，查找起来也很便捷；有时候看我的 java.exe、git.exe 等在什么目录啊，也搜索一下(不要和我说环境变量，我自己配置的我会不知道？只是那样烦琐)。

**Tip: 设置一个快捷键将更效率：选择 everything 快捷方式 > 鼠标右键 > 属性(菜单) > 快捷方式(tab) > 快捷键(输入框) > 设置自己的快捷键(我的：Ctrl + Alt + e)** _上面的步骤不需要我截图吧，需要的自行放弃，我想还是鼠标适合你。_

## cmder

链接：https://cmder.net/

耍 Linux 的大佬略过，用 Mac 的土豪请走开。

这是一个 windows 下增强版的 cmd 命令行工具。美观而强大。看起来很 sexy（官网说的，不是我说的）。

cmder 分 mini 版和 full 版，软件大小不一样，功能也不一样。我用的 mini 版，常用一些 Linux 的命令行，grep，cat，less，curl 等等，在里面用 IPython 也觉得更赏心悦目一点。

有人说我应该用 powershell 啊，或 Linux 子系统啊(WSL)，我都用啊，不冲突，定位不一样。

优化配置文章可以参考：

https://zhuanlan.zhihu.com/p/28400466

https://www.jeffjade.com/2016/01/13/2016-01-13-windows-software-cmder/

## switcheroo

链接：https://github.com/kvakulo/Switcheroo/releases

软件列表切换，和 win 自带的 alt+tab 类似。

但我更喜欢这个，它可以在软件列表输入关键字过滤，可以很方便快捷地在软件间切换。

## Vim

链接：https://www.vim.org/download.php

你除了可以享受到一个伟大的编辑器，还可以帮助到乌干达小朋友。

Vim 门槛有点高，如果你只需要一个简单的替换 notepad 的编辑器，可以移步下一个软件：Notepad++。

如果你有兴趣有毅力学习一个编辑器（打造一下，成为一个 IDE 也是可以的），来提升文本编辑的效率，不妨接触一下。本人文本编辑，IDEA，Chrome 都通过插件 Vim 化了，上手则离不开(摊手)。

## Notepad++

链接：https://notepad-plus-plus.org/downloads/

win 自带文本编辑器增强版。

语法高亮，列编辑，支持插件，行号显示，隐藏字符显示，文本查找替换(支持正则)，文件编码修改等。

## ScreenToGif

链接：https://www.screentogif.com/?l=zh_cn

一个小巧的录屏软件。

录制和编辑 Gif。如果你可以用 Gif 来向别人展示一些操作，是不是更 nice，更易于理解？这款录屏软件小巧而实用，值得推荐。

## Beyond compare

链接：https://www.scootersoftware.com/

文件夹和文件对比工具。专业级的对比，精确到词语级别的对比。

_我的配置和你的一样啊，为什么在我这里不行？来先把你配置文件发我看看，我一对比，你个憨批，这里多个字符啊。_ 这只是文件对比的一个场景而已，在我的桌面上有两个文件(diff1.txt，diff2.txt)，就是我经常用来对比文件用的，这样就不用每次都新建文件，又懒了一次。

（澄清一下，我的电脑桌面是很干净的，绝对不是爬满文件的那种；diff1，diff2(对比用的)，临时.txt(记录临时的东西的)，日志.md(每天工作日志)，workspace(按项目或需求分的文档目录)，桌面就这样，再加一张大气上档次的背景图，仅此而已矣。）

## Postman(Postwoman)

链接：https://www.postman.com/

一款 HTTP 图形化客户端。

做 Web 开发的肯定离不开它。测试接口必用。

基础功能我就不介绍，大家都用，我讲一些我常用的功能：

1. cookies 同步(可以在 chrome 上先登录，在 postman 上就可以同步 cookies，这样就可以访问需要登陆的接口)。

2. environment 和 variables 配置(我会为不同的项目和环境配置不同的 environment 并配置一些参数，如 host，这样同一个接口我不需要为不同的环境创建多个)。

3. Pre-script request(Postman 支持在发送请求前先运行一段 javascript 脚本)，这里举例一个我常用的：

   ```javascript
   var appId = 'xxx'
   var appKey = 'xxx'
   var timestamp = new Date().getTime();
   var nonce = Math.random().toString(36).substring(2, 10)

   pm.environment.unset("appId");
   pm.environment.set("appId", appId);

   pm.environment.unset("timestamp");
   pm.environment.set("timestamp", timestamp);

   pm.environment.unset("nonce");
   pm.environment.set("nonce", nonce);

   //MD5加密签名规格，并赋值给环境变量`sign`
   var context = appId + timestamp + nonce + appKey
   pm.environment.unset("sign");
   pm.environment.set("sign", CryptoJS.MD5(context).toString());
   ```

   看出来没有，在测试一些有 clientSecret 校验的接口，每次手动生成 sign 实在是反人类。有了这段脚本，你就可以忘记这个事了。_（关于 pre-script 大家有兴趣可以去找点资料，可以的话我之后出一篇博客详细讲讲，并分享一些我常用的脚本）。_

## Surfingkeys(或 vimium)

链接：surfingKeys(https://github.com/brookhong/Surfingkeys)

链接：vimium(https://github.com/philc/vimium)

The hacker's browser.

兼具效率与装逼，像极客一样上网。这是一款 chrome 插件(Surfingkeys 是国人开发的一个增强版)，让你可以使用 vim 常用快捷方式操作 chrome 浏览器。体验不用鼠标的上网方式。

打开网页，切换标签，网页后退前进，mark，搜索打开书签，网页滚动翻页，快速复制当前 URL，快速搜索粘贴板内容。如此这些都可以在 partner 们瞠目结舌的表情下敲击几下键盘完成。

我在乎的关键还是效率，是的效率。没有别的。

## PlantUML

链接：https://plantuml.com/zh/

像写代码一样画图。

UML 对于技术文档来说，真的很重要。无论是为了加深自己的理解还是更友好的展示交流。

UML 工具很多，rose，startUML，visio。我常用的是在线版的[draw.io](https://app.diagrams.net/)。PlantUML 可以画几乎所有的 UML 图，不过我用它一般画的最多的是`时序图` 和`流程图`。最近发现其又增加了思维导图的特性，还兼容 Markdown 语法。看了一遍，做一些简单的思维导图是没问题的。

如果读者感兴趣。可以单拎出来细讲一下。

## draw.io

链接：https://app.diagrams.net/

我常用的一个 UML 在线画图工具。具体不详讲。有兴趣可以体验一下。支持本地，google drive，onedrive 存储。

## Typora

Markdown 编辑器。

一个程序员必须要会写文档，有时候文档比代码重要。而写文档最推荐 Markdown 语法，首先语法简单，聪明的你们半小时入门，一天就可以六六六了。其次很多博客和文档平台都支持 Markdown 语法，一招降龙十八掌打遍天下。

懂了 Markdown 语法，你还需要什么？没错，一款好用的 md 编辑器。

Typora 支持各种主题，支持即写即渲染，支持导出各种文件（我很多接口文档都是通过它导出的 pdf 给第三方）。

其他不多说，本文就是在 Typora 中完成的。

## XMind

链接：https://www.xmind.cn/

记录你的想法。

思维导图是东尼·博赞提出的一种记录笔记、思维、想法的方法。以此而催生了一批思维导图软件。各款都有自己的优点和缺点，在此我不向大家推荐软件。加上这条，其实是推荐思维导图这种记录方式，无论你用什么软件，记下了，拓展了思维才是正确的。

## Captura

## 最后

我的开发系统是 win，Mac 用户和 Linux 用户没帮助到，抱歉（我是软粉，巨硬最强）。

另外，没错，我是 Vim 党，Emacs 党走开。

本文每款软件只是推荐，所以只有简单的说明。详细的安装和使用说明请看下面。

> 如果对以上一两款有兴趣，可以自行找资料了解(推荐官网)，也可以留言，有时间可以详细分享一下我的使用经验。如果你有其他上面没提到的小工具，不妨在评论区一起分享。请关注我的公众号。

![MageByte](https://magebyte.oss-cn-shenzhen.aliyuncs.com/wechat/Snip20200314_5.png)
