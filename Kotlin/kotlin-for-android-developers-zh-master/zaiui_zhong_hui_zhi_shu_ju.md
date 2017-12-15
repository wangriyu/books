# 在UI中绘制数据

`MainActivity`中的代码有些小的改动，因为现在有真实的数据需要填充到adapter中。异步调用需要被重写成：

```kotlin
async() {
	val result = RequestForecastCommand("94043").execute()
	uiThread{
		forecastList.adapter = ForecastListAdapter(result)
	}
}
```

Adapter也需要被修改：

```kotlin
class ForecastListAdapter(val weekForecast: ForecastList) :
        RecyclerView.Adapter<ForecastListAdapter.ViewHolder>() {
        
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int):
            ViewHolder? {
        return ViewHolder(TextView(parent.getContext()))
    }
        
    override fun onBindViewHolder(holder: ViewHolder,
            position: Int) {
        with(weekForecast.dailyForecast[position]) {
            holder.textView.text = "$date - $description - $high/$low"
	} 
}
    override fun getItemCount(): Int = weekForecast.dailyForecast.size
    
    class ViewHolder(val textView: TextView) : RecyclerView.ViewHolder(textView)
}
```

> __with函数__
> > with是一个非常有用的函数，它包含在Kotlin的标准库中。它接收一个对象和一个扩展函数作为它的参数，然后使这个对象扩展这个函数。这表示所有我们在括号中编写的代码都是作为对象（第一个参数）的一个扩展函数，我们可以就像作为this一样使用所有它的public方法和属性。当我们针对同一个对象做很多操作的时候这个非常有利于简化代码。

在这一章中有很多新的代码加入，所以检出[库中]的代码吧。

[库中]: https://github.com/antoniolg/Kotlin-for-Android-Developers