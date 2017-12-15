# 创建一个layout

显示天气预报的列表我们使用`RecyclerView`，所以你需要在`build.gradle`中增加一个新的依赖：

```groovy
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile "com.android.support:appcompat-v7:$support_version" 
    compile "com.android.support:recyclerview-v7:$support_version" ...
}
```

然后，`activity_main.xml`如下：

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
             android:layout_width="match_parent"
             android:layout_height="match_parent">
    <android.support.v7.widget.RecyclerView
        android:id="@+id/forecast_list"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</FrameLayout>
```

在`Mainactivity.kt`中删除掉之前用来测试的能正常运行的所有代码（现在应该会提示错误）。暂且我们使用老的`findViewByid()`的方式：

```kotlin
val forecastList = findViewById(R.id.forecast_list) as RecyclerView
forecastList.layoutManager = LinearLayoutManager(this)
```

如你所见，我们定义类一个变量并转型为`RecyclerView`。这里与Java有点不同，我们会在下一章分析这些不同之处。`LayoutManager`会通过属性的方式被设置，而不是通过setter，这个layout已经足够显示一个列表了。

>对象实例化
>>对象实例化也是与Java中有些不同。如你所见，我们去掉了`new`关键字。这时构造函数仍然会被调用，但是我们省略了宝贵的四个字符。`LinearLayoutManager(this)`创建了一个对象的实例。