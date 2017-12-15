# 安装Kotlin插件

~~IDE它本身并不能理解Kotlin。就像前面部分讲到，它是为Java开发设计的。但是Kotlin团队创建了一系列强大的插件让我们更轻松地实现。前往Android Studio的`Preferences`中`Plugin`栏，然后安装如下两个插件：~~
- ~~Kotlin：这是一个基础的插件。它能让Android Studio懂得Kotlin代码。它会每次在新的Kotlin语言版本发布的时候发布新的插件版本，这样我们可以通过它发现新版本特性和弃用的警告。这是你要使用Kotlin编写Android应用唯一的插件。但是我们现在还需要另外一个。~~
- ~~Kotlin Android Extensions：Kotlin团队还为Android开发发布了另外一个有趣的插件。这个Android Extensions可以让你自动地从XML中注入所有的View到Activity中，举个例子，你不需要使用`findViewById()`。你将会立即得到一个从属性转换过来的view。你将需要安装这个插件来使用这个特性。我们会在下一章中深入地去讲解这个。~~

因为从Intellij 15开始，插件是被默认安装了的，但是你的Android Studio可能并没有。所以你需要进入Android Studio 的Preferences的plugin栏，然后安装Kotlin插件。如果你不会就去搜索下。

现在我们的环境已经可以理解Kotlin语言了，可以就像我们使用Java一样无缝地编译它，执行它。