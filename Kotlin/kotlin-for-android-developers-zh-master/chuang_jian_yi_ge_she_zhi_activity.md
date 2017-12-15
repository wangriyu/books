# 创建一个设置activity

当toolbar上溢出菜单（`overflow menu`）的`settings`选项被点击时，需要打开一个新的Activity。所以首先要做的事情时需要一个新的`SettingActivity`：

```kotlin
class SettingsActivity : AppCompatActivity() {
	
	override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_settings)
        setSupportActionBar(toolbar)
        supportActionBar.setDisplayHomeAsUpEnabled(true)
    }

	override fun onOptionsItemSelected(item: MenuItem) = when (item.itemId) {
        android.R.id.home -> { onBackPressed(); true }
        else -> false
	}
}
```

当用户离开这个界面的时我们需要保存用户`preference`（偏好），所以我们需要像处理`Back`一样处理`Up`动作，重定向动作到`onBackPressed`。现在，让我们要创建一个XML布局。对于这个`preference`来说一个简单`EditText`就足够了：

```xml
<FrameLayout
	xmlns:android="http://schemas.android.com/apk/res/android"
	android:layout_width="match_parent"
	android:layout_height="match_parent">

	<include layout="@layout/toolbar"/>
	<LinearLayout
		android:orientation="vertical"
		android:layout_width="match_parent"
		android:layout_height="match_parent"
		android:layout_marginTop="?attr/actionBarSize"
		android:padding="@dimen/spacing_xlarge">
	
		<TextView
			android:layout_width="wrap_content"
			android:layout_height="wrap_content"
			android:text="@string/city_zipcode"/>
		
		<EditText
			android:id="@+id/cityCode"
			android:layout_width="match_parent"
			android:layout_height="wrap_content"
			android:hint="@string/city_zipcode"
			android:inputType="number"/>

	</LinearLayout>
</FrameLayout>
```

然后只需要在`AndroidManifest.xml`中声明这个activity：

```xml
<activity
android:name=".ui.activities.SettingsActivity"
android:label="@string/settings"/>
```