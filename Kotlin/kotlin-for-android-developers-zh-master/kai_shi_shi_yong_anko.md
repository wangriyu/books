# 开始使用Anko

在之前，让我们来使用Anko来简化一些代码。就像你将看到的，任何时候你使用了Anko库中的某些东西，它们都会以属性名、方法等方式被导入。这是因为Anko使用了__扩展函数__在Android框架中增加了一些新的功能。我们将会在以后看到扩展函数是什么，怎么去编写它。

在`MainActivity:onCreate`，一个Anko扩展函数可以被用来简化获取一个RecyclerView：

```kotlin
val forecastList: RecyclerView = find(R.id.forecast_list)
```

我们现在还不能使用库中更多的东西，但是Anko能帮助我们简化代码，比如，实例化Intent，Activity之间的跳转，Fragment的创建，数据库的访问，Alert的创建……我们将会在实现这个App的过程中学习到很多有趣的例子。