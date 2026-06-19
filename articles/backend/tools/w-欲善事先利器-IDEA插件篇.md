> 工欲善其事，必先利其器，好鞋踢好球是非常合乎逻辑的事情。
>
> ——《长江七号》

同样的开场白，不一样的酒，不一样的故事。

上篇《欲善事先利器——系统篇》已经推荐了一些个人常用的效率系统软件。觉得有帮助的，有共鸣的 Rock 一下。我们继续新篇——IDEA 插件篇。用 Eclipse 的请原谅，本人已经好几年没用过 Eclipse 了，给不了你好的建议。

**以下插件插件直接在 IDEA 插件管理里面搜索安装：**

`IDEA > Ctrl+A > 输入"plugins" > 选择plugins > 选择marketplace(tab) > 输入插件名 > 选择Install`

_其中提供链接的是希望读者自己看一看官方文档。_

## AceJump

快速定位光标，有它，你可以丢掉鼠标了。

你只需要 `Ctrl + ;` 然后输入跳转到的字符即可定位到相应的位置。

![](https://magebyte.oss-cn-shenzhen.aliyuncs.com/toolsscreen2gif.gif)

## IdeaVim

在 Idea 中使用 Vim 风格写代码，只适合 Vim 党。不多介绍，懂得自然懂，不懂的自行先学 Vim(自动狗头)。

## GenerateAllSetter

链接：https://github.com/gejun123456/intellij-generateAllSetMethod

- 为本地变量快速生成 setter 方法。再不需要一个一个去 set 了，经常忘了一个结果就出 bug 了。
- 在方法上快速 convert 的 setter 形式的代码。

## CamelCase

驼峰式大小写切换插件。

可以通过快捷键在 CamelCase, camelCase, snake_case and SNAKE_CASE 之间快速切换。

默认快捷键：`ctrl + shift + u`

![MageByte](https://magebyte.oss-cn-shenzhen.aliyuncs.com/tools/idea_plugins/camelCase.gif)

## Free MyBatis plugin

1. 快速在 Mybatis Mapper 类方法和 Mybatis mapper.xml sql 语句间相互定位。

   ![MageByte](https://magebyte.oss-cn-shenzhen.aliyuncs.com/tools/idea_plugins/free-mybatis-1.gif)

2. 快速根据方法定义生成相应的 mapper 语句。

   ![MateByte](https://magebyte.oss-cn-shenzhen.aliyuncs.com/tools/idea_plugins/free-mybatis-2.gif)

## Codehelper.generator

链接：https://github.com/zhengjunbase/codehelper.generator

**特性：**

- 根据 Pojo 文件一键生成 Dao，Service，Xml，Sql 文件。
- Pojo 文件更新后一键更新对应的 Sql 和 mybatis xml 文件。
- 提供 insert，insertList，update，select，delete 五种方法。
- 能够批量生成多个 Pojo 的对应的文件。
- Pojo 文件新增字段后，同时生成添加字段的 sql 语句。
- 自动将 pojo 的注释添加到对应的 Sql 文件的注释中。
- 丰富的配置，如果没有配置文件，则会使用默认配置。
- 可以在 Intellij Idea 中快捷键配置中配置快捷键。
- 目前支持 MySQL + Java，后续会支持更多的 DB。

## Maven Helper

查看 maven 包引用关系，快速定位有冲突的吧。比起 IDEA 自带的 `Diagrams` 更清晰好用。

![MageByte](https://magebyte.oss-cn-shenzhen.aliyuncs.com/tools/idea_plugins/maven-helper.png)

## CodeMaker

链接：https://github.com/x-hansong/CodeMaker

有点想法的程序员大都会对一直重复的代码很暴躁，想要么能不能通过框架解决，要么能不能通过代码自动生成解决。`CodeMaker`就是一个 IDEA 代码生成插件，你可以根据类来生成相应的 Template（基于 Velocity），之后想生成类似的类就直接可以通过 IDEA 生成了。

![MageByte](https://magebyte.oss-cn-shenzhen.aliyuncs.com/tools/idea_plugins/codeMaker.gif)

## Git Commit Template

Git Commit Message 一定要简约而实用，描述清楚提交的功能。_插一句题外话，注释的老代码就直接删除掉，不要说什么以后可能会用到啊，git history 已经帮你记录了，请不要留在当前版本下！！！_

- 按如下风格整理 message

  ```
  <type>(<scope>): <subject>
  <BLANK LINE>
  <body>
  <BLANK LINE>
  <footer>
  ```

- 按如下方式提交 message

  ![MageByte](https://magebyte.oss-cn-shenzhen.aliyuncs.com/tools/idea_plugins/git-commit-t-3.png)

- message 将看起来如下

  ![MageByte](https://magebyte.oss-cn-shenzhen.aliyuncs.com/tools/idea_plugins/git-commit-t.png)

## Grep Console

链接：https://plugins.jetbrains.com/plugin/7125-grep-console

1. 让 Console 日志有颜色，可以对 trace，debug，info，warn，error 配置不同的颜色。

   ![MageByte](https://magebyte.oss-cn-shenzhen.aliyuncs.com/tools/idea_plugins/gc-1.png)

2. grep 过滤日志

   ![MageByte](https://magebyte.oss-cn-shenzhen.aliyuncs.com/tools/idea_plugins/gc-2.png)

## Jackson Generator Plugin

链接：https://plugins.jetbrains.com/plugin/7678-jackson-generator-plugin

快速在 class 和 json 间相互生成。同样的还有 `Gson Generator`。一个生成 `Jackson` 风格的类(注解)，一个生成 `Gson` 风格的类。

## Lombok

链接：https://plugins.jetbrains.com/plugin/6317-lombok

使用 Lombok 必须安装的插件。

Lombok 通过添加注解的方式来生成 getter，setter，toString，builder 等这些无意义代码（原理是字节码修改，maven 插件和 idea 插件）。

## Rainbow Brackets

链接：https://plugins.jetbrains.com/plugin/10080-rainbow-brackets

让你的左括号和对应的右括号(大小括号都可以)显示相同的颜色，以此快速看出括号的范围。

like this:

![MageByte](https://magebyte.oss-cn-shenzhen.aliyuncs.com/tools/idea_plugins/rb.png)

## String Manipulation

链接：https://plugins.jetbrains.com/plugin/2162-string-manipulation

和 `CamelCase` 的功能类似，不过除了 `camel` 风格字符串转换，还包括很多强大的功能：

- 风格切换(camelCase, kebab-lowercase, KEBAB-UPPERCASE, snake_case, SCREAMING_SNAKE_CASE, dot.case, words lowercase, First word capitalized, Words Capitalized, PascalCase)。
- Un/Escape 代码(Java、JavaScript、SQL、HTML 等)。
- 编码/解码(MD5、Hex、Base64 等)
- 排序字符行

![MageByte](https://magebyte.oss-cn-shenzhen.aliyuncs.com/tools/idea_plugins/sm-1.png)

![MageByte](https://magebyte.oss-cn-shenzhen.aliyuncs.com/tools/idea_plugins/sm-2.png)

**以上插件建议直接在 IDEA 插件管理里面搜索安装：**

`IDEA > Ctrl+A > 输入"plugins" > 选择plugins > 选择marketplace(tab) > 输入插件名 > 选择Install`

_其中提供链接的是希望读者自己看一看官方文档。_

**推荐：**

1. [《欲善事先利器——系统篇》](http://mp.weixin.qq.com/s?__biz=MzU3NDkwMjAyOQ==&mid=2247483975&idx=1&sn=6f51a57582f1c965b747fc157253fdeb&chksm=fd2a1825ca5d9133f938f94550335cb79aba324b1cb61d422f4821c0e5a57dc44196f7e4b656&scene=21#wechat_redirect)
2. 《欲善事先利器——IDEA 插件篇》(本篇)
3. 《欲善事先利器——Library 篇》（待更新）
4. 《欲善事先利器——流程篇》（待更新）
5. 《欲善事先利器——网站篇》（待更新）

> 如果对以上一两款插件有兴趣，可以自行找资料了解(推荐官网)，也可以留言，有时间可以详细分享一下我的使用经验。如果你有其他上面没提到的小工具，不妨在评论区一起分享。请关注我的公众号。

![MageByte](https://magebyte.oss-cn-shenzhen.aliyuncs.com/wechat/Snip20200314_5.png)
