# The Recycler Adapter

我们同样需要一个`RecyclerView`的Adapter。[之前我在我博客中讨论过`RecyclerView`]，如果你以前没有使用过，它可能会有所帮助。

`RecyclerView`中所使用到的布局现在只需要一个`TextView`，我会手动去创建这个简单的文本列表。增加一个名为`ForecastListAdapter.kt`的Kotlin文件，包括如下代码：
```kotlin
class ForecastListAdapter(val items: List<String>) :
        RecyclerView.Adapter<ForecastListAdapter.ViewHolder>() {
        
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        return ViewHolder(TextView(parent.context))
    }
    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.textView.text = items.[position]
    }
    
    override fun getItemCount(): Int = items.size
    
    class ViewHolder(val textView: TextView) : RecyclerView.ViewHolder(textView)
}
```

又是如此，我们可以像访问属性一样访问context和text。你可以保持以往那样操作（使用getters和setters）,但是你会得到一个编译器的警告。如果你还是倾向于Java中的使用方式，这个检查可以被关闭。但是一旦你使用上了这种属性调用的方式你就会爱上它，而且它也节省了额外的字符总量。

回到`MainActivity`，现在简单地创建一系列的String放入List中，然后使用创建分配Adapter实例。

```kotlin
private val items = listOf(
    "Mon 6/23 - Sunny - 31/17",
    "Tue 6/24 - Foggy - 21/8",
    "Wed 6/25 - Cloudy - 22/17",
    "Thurs 6/26 - Rainy - 18/11",
    "Fri 6/27 - Foggy - 21/10",
    "Sat 6/28 - TRAPPED IN WEATHERSTATION - 23/18",
    "Sun 6/29 - Sunny - 20/7"
    )
    
override fun onCreate(savedInstanceState: Bundle?) {
    ...
    val forecastList = findViewById(R.id.forecast_list) as RecyclerView
    forecastList.layoutManager = LinearLayoutManager(this) 
    forecastList.adapter = ForecastListAdapter(items)
}
    
```

>List的创建
>>尽管我会在本书后面来对Collection进行讲解，但是我现在仅仅简单地解释你可以通过使用一个函数`listOf`创建一个常量的List（很快我们就会讲到的`immutable`）。它接收一个任何类型的`vararg`（可变长的参数），它会自动推断出结果的类型。

>> 还有很多其它的函数可以选择，比如`setOf`，`arrayListOf`或者`hashSetOf`。

为了优化项目的结构，我也移动了一些类到新的包里面。

我们在上面很简短的代码中看到了很多新的东西，所以我会在下一章讲到它们。我们现在必须要学习一些比如基本类型，变量，属性等比较重要的概念才能继续下去。



[之前我在我博客中讨论过`RecyclerView`]: http://antonioleiva.com/recyclerview/