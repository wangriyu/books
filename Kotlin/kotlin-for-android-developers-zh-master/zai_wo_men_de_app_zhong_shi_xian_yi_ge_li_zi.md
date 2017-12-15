# 在我们的App中实现一个例子

接口可以被用来从类中提取出相似行为的通用代码。比如，我们可以创建一个接口用于处理app的toolbar。`MainActivity`和`DetailActivity`在处理`toolbar`时会共享这些相似的代码。

但是受限，我们需要做出一些改变，使用被定义在布局中`toolbar`，而不是标准的`ActionBar`。第一件事是继承`NoActionBar`主题。这样`toolbar`不会自动被包含进来：

```kotlin
<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
	<item name="colorPrimary">#ff212121</item>
	<item name="colorPrimaryDark">@android:color/black</item>
</style>
```

我们使用`light`主题。然后我们创建一个`toolbar`的布局，我们稍后会在其它的布局中使用到它：

```kotlin
<android.support.v7.widget.Toolbar
	xmlns:app="http://schemas.android.com/apk/res-auto"
	xmlns:android="http://schemas.android.com/apk/res/android"
	android:id="@+id/toolbar"
	android:layout_width="match_parent"
	android:layout_height="?attr/actionBarSize"
	android:background="?attr/colorPrimary"
	app:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
	app:popupTheme="@style/ThemeOverlay.AppCompat.Light"/>
```

`toolbar`指定了它自己的背景，一个针对自己的`dark`主题和一个针对生成的弹出框的`light`主题（`overflow menu`实例）。我们现在已经有了相同的主题：`light`主题和`dark`主题的`Action Bar`。

下一步我们将修改`MainActivity`的布局，增加一个toolbar：

```kotlin
<FrameLayout
	xmlns:android="http://schemas.android.com/apk/res/android"
	android:layout_width="match_parent"
	android:layout_height="match_parent">

	<android.support.v7.widget.RecyclerView
		android:id="@+id/forecastList"
		android:layout_width="match_parent"
		android:layout_height="match_parent"
		android:clipToPadding="false"
		android:paddingTop="?attr/actionBarSize"/>
	
	<include layout="@layout/toolbar"/>
</FrameLayout>
```

现在toolbar被增加到布局中，我们可以开始使用它。我们创建了一个接口，它可以让我们：

- 改变title
- 指定是否显示上一步的导航动作
- 滚动时的toolbar动画
- 给所有的activity设置相同的菜单，甚至行为

然后让我们定义`ToolbarManager`：

```kotlin
interface ToolbarManager {
    val toolbar: Toolbar
    ...
}
```

它将需要一个toolbar属性。接口是无状态的，所以属性可以被定义，但是不能赋值。子类会实现这个接口并重写这个属性。

另一方面，我们可以不使用重写来实现无状态的属性。也就是说属性不需要维护一个`backup field`。一个处理toolbar title属性的例子：

```kotlin
var toolbarTitle: String
    get() = toolbar.title.toString()
    set(value) {
        toolbar.title = value
    }
```

因为属性仅仅使用了toolbar，它不需要保存任何新的状态。

我们现在创建了一个新的函数用来初始化toolbar，inflate一个menu并且设置一个listener：

```kotlin
fun initToolbar(){
	toolbar.inflateMenu(R.menu.menu_main)
    toolbar.setOnMenuItemClickListener {
		when (it.itemId) {
		    R.id.action_settings -> App.instance.toast("Settings")
		    else -> App.instance.toast("Unknown option")
		}
		true
	}
}
```

我们可以增加一个函数用来开启toolbar上面导航icon，设置一个箭头的icon并设置一个当icon被按压时触发的事件：

```kotlin
fun enableHomeAsUp(up: () -> Unit) {
       toolbar.navigationIcon = createUpDrawable()
       toolbar.setNavigationOnClickListener { up() }
}

private fun createUpDrawable() = with (DrawerArrowDrawable(toolbar.ctx)){
    progress = 1f
	this
}
```

这个函数接收一个listener，使用[DrawerArrowDrawable]来创建一个最后状态（当箭头已经显示时）的drawable，然后把listener设置给toolbar。

最后，接口将会提供一个函数，它允许toolbar可以attached到一个scroll上面，并且根据scroll的方向来执行动画。当往下滚动时toolbar会消失 ，往上滚动toolbar会再次显示：

```kotlin
fun attachToScroll(recyclerView: RecyclerView) {
    recyclerView.addOnScrollListener(object : RecyclerView.OnScrollListener() {
		override fun onScrolled(recyclerView: RecyclerView?, dx: Int, dy: Int) {
			if (dy > 0) toolbar.slideExit() else toolbar.slideEnter()
	    }
    })
}
```

我们会创建两个用于view从屏幕中显示或者消失动画的扩展函数。我们会检查是否动画之前没有执行过。这种方式可以避免每次不同的滚动view都会执行动画：

```kotlin
fun View.slideExit() {
	if (translationY == 0f) animate().translationY(-height.toFlat())
}

fun View.slideEnter() {
	if (translationY < 0f) animate().translationY(0f)
}
```

在`toobar manager`实现之后，是时候在`MainActivity`中使用它了。我们首先指定toolbar属性。我们可以使用`lazy`委托实现，这样会在我们第一次使用它的时候才会inflate：

```kotlin
override val toolbar by lazy { find<Toolbar>(R.id.toolbar) }
```

`MainActivity`将会仅仅初始化toolbar并attach到`RecyclerView`的滚动并修改toolbar的title：

```kotlin
override fun onCreate(savedInstanceState: Bundle?) { 
super.onCreate(savedInstanceState) 
setContentView(R.layout.activity_main) initToolbar()
    forecastList.layoutManager = LinearLayoutManager(this)
    attachToScroll(forecastList)
    async {
        val result = RequestForecastCommand(94043).execute()
        uiThread {
			val adapter = ForecastListAdapter(result) {
		    startActivity<DetailActivity>(DetailActivity.ID to it.id,
			            DetailActivity.CITY_NAME to result.city)
			}
			forecastList.adapter = adapter
			toolbarTitle = "${result.city} (${result.country})"
		} 
	}
}
```

`DetailActivity`也需要一些布局上的修改：

```kotlin
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    
    <include layout="@layout/toolbar"/>
    
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:gravity="center_vertical"
		android:paddingTop="@dimen/activity_vertical_margin"
        android:paddingLeft="@dimen/activity_horizontal_margin"
        android:paddingRight="@dimen/activity_horizontal_margin"
        tools:ignore="UseCompoundDrawables">
       ....
    </LinearLayout>
    
</LinearLayout>
```

使用相同的方式去指定toolbar属性。`DetailActivity`也会初始化toolbar，设置title并且开启导航返回icon：

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_detail)
    
	initToolbar()
	toolbarTitle = intent.getStringExtra(CITY_NAME) 
	enableHomeAsUp { onBackPressed() }
	...
}
```

接口可以帮助我们从类中提取出公共的代码来共享相似的行为。可以作为让我们代码精炼合理简洁可复用的替代方案。思考哪方面接口可以帮助你写出更好的代码。

[DrawerArrowDrawable]: https://developer.android.com/reference/android/support/v7/graphics/drawable/DrawerArrowDrawable.html