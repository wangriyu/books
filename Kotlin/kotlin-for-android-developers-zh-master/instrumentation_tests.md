# Instrumentation tests

`Instrumentation tests`有一点不同。它们通常被使用在UI交互上，我们需要一个应用程序实例跑的同时执行测试。达到这个，我们就需要在设备上部署并运行。

这类的测试必须要放在`androidTest`文件夹中，我们必须要修改`Build Variants`区域的`Test Artifact`为`Android Instrumentation Tests`。实现instrumentation的官方库是[Espresso]，它通过`Actions`、`filter`以及检测结果的`ViewMatchers`和`Matchers`可以帮助我们更简单地使用。

配置比之前更加难一点。我们需要下载额外的库和`Gradle`的配置。好事是Kotlin的测试不需要添加额外的东西，所以如果你已经知道怎么去配置`Espresso`，它将对你来说是很简单的。

首先，在`defaultConfig`中指定`test runner`：

```groovy
defaultConfig {
    ...
    testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
}
```

当你处理完该runner，然后增加其它的依赖，这次是用`androidTestCompile`。这种方式，这些库只会再编译运行`instrumentation tests`的时候才被增加：

```groovy
dependencies {
    ...
    androidTestCompile "com.android.support:support-annotations:$support_version"
    androidTestCompile "com.android.support.test:runner:0.4.1"
    androidTestCompile "com.android.support.test:rules:0.4.1"
    androidTestCompile "com.android.support.test.espresso:espresso-core:2.2.1"
    androidTestCompile ("com.android.support.test.espresso:espresso-contrib:2.2.1"){
	    exclude group: 'com.android.support', module: 'appcompat'
		exclude group: 'com.android.support', module: 'support-v4'
		exclude module: 'recyclerview-v7'
}
```

我不想花大量的时间去讲这些，但是这里有为什么需要这些库的简短原因：

- `support-annotations`：其它库中需要使用到。
- `runner`：这是`test runner`，就是我们再`defaultConfig`中指定的那个。
- `rules`：包括一些测试inflate启动activity的规则。我们将会在我们的例子中使用这些规则。
- `espresso-core`：`Espresso`的基本实现，它让`instrument tests`更加容易。
- `espresso-contrib`：它增加了其它额外的功能，比如支持`RecyclerView`测试。我们不得不排除掉一些它的依赖，因为我们已经在这个项目中使用到了，否则测试会出错。

我们现在来创建一个简单的例子。测试将会点击forecast列表的第一行，然后它会判断是否能找到一个id为`R.id.weatherDescription`的view。这个view是在`DetailActivity`中的，这表示我们在测试在RecyclerView里面点击后是否可以成功地导航到详情页面。

```kotlin
class SimpleInstrumentationTest {

	@get:Rule
    val activityRule = ActivityTestRule(MainActivity::class.java)
    
    ...
}
```

首先我们需要指定它运行时使用`AndroidJUnit4`。然后，创建一个activity规则，它会实例化一个测试需要的activity。在Java中，你可以使用`@Rule`。但是如你所知，字段和属性是不一样的，所以如果你像那样去使用的话，执行会失败因为访问属性中的字段是不是public的。你需要加注解的是在getter上面。Kotlin允许指定`get`或者`set`在Rule的名字前面。在这个例子中，只要些`@get:Rule`。

之后，我们已经准备好创建第一个测试了：

```kotlin
@Test fun itemClick_navigatesToDetail() {
    onView(withId(R.id.forecastList)).perform(
            RecyclerViewActions
                .actionOnItemAtPosition<RecyclerView.ViewHolder>(0, click()))
    onView(withId(R.id.weatherDescription))
           .check(matches(isAssignableFrom(TextView::class.java)))
}
```

函数加上了`@Test`注解，这根我们使用`unit test`的方式一样。我们可以开始在测试体中使用`Espresso`。它首先在`RecyclerView`的第一个position中执行了一个点击。然后它检测是否可以找到一个指定id的view且这个view是一个`TextView`。

要运行这个测试，点击顶部的`Run configurations`下拉选择`Edit Configurations...`按下`+`图标，选择`Android Tests`，然后选择`app`模块。现在，在`target device`中选择你喜欢的`target`。点击`OK`然后运行。你应该可以看到App是怎样在你的设备中开始的，它会测试第一个position，打开详情页面然后再次关闭app。

现在我们要做一个更加复杂一点的事情。测试会从toolbar众打开一个溢出菜单，点击`settings`栏，改变城市的`code`，然后检测toolbar的标题是否改变成了对应的标题。

```kotlin
@Test fun modifyZipCode_changesToolbarTitle() {
	openActionBarOverflowOrOptionsMenu(activityRule.activity)
	onView(withText(R.string.settings)).perform(click())
	onView(withId(R.id.cityCode)).perform(replaceText("28830"))
	pressBack()
	onView(isAssignableFrom(Toolbar::class.java))
	        .check(matches(
	            withToolbarTitle(`is`("San Fernando de Henares (ES)"))))
}
```

这个测试实际做的事情：

- 它首先使用`openActionBarOverflowOrOptionsMenu`打开溢出菜单。
- 然后它根据`Settings`文本查找一个view，然后点击这个它。
- 之后，设置界面就会被打开，所以它会查找一个`EditText`并且替换成一个新的`code`。
- 它会点击返回按钮。它会把新的值保存在preferences中，然后关闭Activity。
- 因为`MainActivity`的`onResume`会调用，请求会再调用一次。这时它会获取到新城市的forecast。
- 最后一行将会检测Toolbar我们看到的title是否是新的城市的title。

这不是一个toolbar的title的默认匹配器，但是`Espresso`是很容易扩展的，所以我们可以创建一个新的matcher来实现检测：

```kotlin
private fun withToolbarTitle(textMatcher: Matcher<CharSequence>): Matcher<Any> =
        object : BoundedMatcher<Any, Toolbar>(Toolbar::class.java) {

	override fun matchesSafely(toolbar: Toolbar): Boolean {
		return textMatcher.matches(toolbar.title)
	}

	override fun describeTo(description: Description) {
		description.appendText("with toolbar title: ")
		textMatcher.describeTo(description)
	}                
}
```

`matchSafely`函数是我们检测的地方，而`describeTo`函数为matcher增加了一些新的信息。

这章特别有趣，因为我们看到了在Kotlin中怎么样去完美和谐地整合测试，它们可以没有任何问题地整合测试。查看代码然后你自己运行一下吧。


[Espresso]: https://google.github.io/android-testing-support-library/