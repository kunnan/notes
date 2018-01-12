**Gradle 从入门到实战②全面理解 Gradle-执行时序**

[TOC]

# 什么是 Gradle？

* 一个像 Ant 一样的非常灵活的通用构建工具 
* 一种可切换的, 像 maven 一样的基于合约构建的框架 
* 支持强大的多工程构建 
* 支持强大的依赖管理(基于 ApacheIvy ) 
* 支持已有的 maven 和 ivy 仓库 
* 支持传递性依赖管理, 而不需要远程仓库或者 pom.xml 或者 ivy 配置文件 
* 优先支持 Ant 式的任务和构建 
* 基于 groovy 的构建脚本 
* 有丰富的领域模型来描述你的构建

# 如何学习 Gradle？

* 学习 Groovy（http://docs.groovy-lang.org/）
* 学习 Gradle DSL（https://docs.gradle.org/current/javadoc/org/gradle/api/Project.html）
* 学习 Android DSL 和 Task（http://google.github.io/android-gradle-dsl/current/index.html）

# 使用Gradle wrapper

如果你本地安装了Gradle，那么你就可以使用 gradle 命令来直接构建。如果本地没有安装，那么可以通过 gradle wrapper 来构建，Linux 和 MAC 使用 ./gradlew，而 Windows 上面则使用 gradlew，还可以在 gradle/gradle-wrapper.properties 中配置 Gradle 版本。

# Gradle 脚本的执行时序

Gradle脚本的执行分为三个过程：

* 初始化 

分析有哪些module将要被构建，为每个 module 创建对应的  project 实例。这个时候 settings.gradle 文件会被解析。

* 配置

处理所有的模块的 build 脚本，处理依赖，属性等。这个时候每个模块的 build.gradle 文件会被解析并配置，这个时候会构建整个task 的链表（这里的链表仅仅指存在依赖关系的 task 的集合，不是数据结构的链表）。

* 执行

根据 task 链表来执行某一个特定的 task，这个 task 所依赖的其他 task 都将会被提前执行。

下面我们根据一个实际的例子来详细说明。这里我们仍然采用 VirtualAPK 这个开源项目来做演示，它的地址是：https://github.com/didi/VirtualAPK。

我们以它的宿主端为例，宿主端有如下几个模块：

![](http://mmbiz.qpic.cn/mmbiz_png/zKFJDM5V3Ww5cNmuhdv1dlgVpTFFKfNUTm4z0W3FWG8NHbYARVfj1YicXn0KM0UR0EZB5K7DzWibfVibbKoCibV0Hw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

其中buildSrc是virtualapk-gradle-plugin，为了便于调试我将其重命名为 buildSrc。他们的依赖关系如下：

![](http://mmbiz.qpic.cn/mmbiz_png/zKFJDM5V3Ww5cNmuhdv1dlgVpTFFKfNU5yEyDevLx3bJtAcVknrf9DXSdqh7h0rENBjoGuncdKFiaKyiasKr4avg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

解释一下，app模块依赖 CoreLibrary 和 buildSrc，CoreLibrary 又依赖 AndroidStub。为了大家更好理解，下面加一下log。

![](https://mmbiz.qpic.cn/mmbiz_png/zKFJDM5V3Ww5cNmuhdv1dlgVpTFFKfNU04l3fD0ibiahvgSL5S2hM49F0afiag25EhcF1FL8oGkHWecEmxDDrj0ng/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![](https://mmbiz.qpic.cn/mmbiz_png/zKFJDM5V3Ww5cNmuhdv1dlgVpTFFKfNUdibcNWTnXIwIAotG24PA6wiariauHycRXMh6HIiaJTwaoHows8kwfrVEIw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![](https://mmbiz.qpic.cn/mmbiz_png/zKFJDM5V3Ww5cNmuhdv1dlgVpTFFKfNU9YFeyicfSZ53b9Dib6TpWRQsWfCWJfaFAoDMhDGm0Oe5lWBnibj7nqhjw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

![](https://mmbiz.qpic.cn/mmbiz_png/zKFJDM5V3Ww5cNmuhdv1dlgVpTFFKfNUGXrItkibPUZPwIJG35gXzTSrpKiarD37AeFG189EkKs3wF7mc4EmnhRg/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

现在随便执行一个 task，比如./gradlew clean，那么将会输出如下日志，大家对比着日志，应该能明白Gradle脚本的执行顺序了吧。

![](https://mmbiz.qpic.cn/mmbiz_png/zKFJDM5V3Ww5cNmuhdv1dlgVpTFFKfNUJBQSibXMVm2F6kouIJUgzyVo2SLia3fqPMdMK2fzcYrsuHN46vqafauw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

可以看到，Gradle执行的时候遵循如下顺序： 

1. 首先解析settings.gradle来获取模块信息，这是初始化阶段； 
2. 然后配置每个模块，配置的时候并不会执行task； 
3. 配置完了以后，有一个重要的回调 project.afterEvaluate，它表示所有的模块都已经配置完了，可以准备执行task了； 
4. 执行指定的task。

如果注册了多个project.afterEvaluate回调，那么执行顺序等同于注册顺序。在上面的例子中，由于buildSrc中的回调注册较早，所以它也先执行。


