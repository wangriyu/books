# Unit testing

我不会对`unit testing`（单元测试）是什么的话题展开讨论。存在很多定义，但是都有一些细微的不同。一个普通的观点可能是`unit testing`验证一个单位（`unit`）的源代码的测试。一个单位（`unit`）包含什么就留给读者了。在我们的例子中，我仅仅去定义了一个`unit test`作为一个不需要设备运行的测试。IDE将会运行这些测试然后显示最后的结果分辩哪些测试成功哪些测试失败了。

`Unit testing`通常使用`JUnit`库。所以让我们增加这个依赖到`build.gradle`。因为这个依赖只会在跑测试的时候才会用到，所以我们可以使用`testCompile`而不是`compile`。用这种方式，这个库会在正式编译时忽略掉，可以减少APK的大小：

```groovy
dependencies {
	...
	testCompile 'junit:junit:4.12'
}
```

现在同步gradle来获取该库并加入到你的项目中。为了开启`unit testing`，打开`Build Variants`tab（你可能可以在IDE的左边找到它），点击`Test Artifact`下拉，你应该选择`Unit Tests`。

另一件你需要做的事情是创建一个新的文件夹。在src下面，你可能已经有`androidTest`和`main`了。创建另一个名为`test`的文件夹，再在它下面创建一个`java`文件夹。所以现在你应该有一个名为`src/test/java`绿色的文件夹。这是IDE发现我们在使用`Unit Test`模式好的迹象，这个文件夹中将会包括一些测试文件。

我们来写一个非常简单的测试来看看一切是不是正常运行了。使用合适的包名（我的是`com.antonioleiva.weatherapp`，但是你需要使用你app中的主包名）创建一个新的名为`SimpleTest`的Kotlin类。当你创建完，编写如下简单的测试：

```kotlin
import org.junit.Test
import kotlin.test.assertTrue
class SimpleTest {
	@Test fun unitTestingWorks() {
	    assertTrue(true)
	}
}
```

使用`@Test`注解来辨别该函数为是一个测试。确认是`org.unit.Test`。然后增加一个简单的断言。它只是判断了true是否是true，它显然会成功。这个测试只是用开确认一切配置正确。

执行测试，只需要在你在`test`下创建的新的`java`文件夹上右击，然后选择`Run All Tests`。当编译完成后，它会运行测试并会看见结果简介的显示。你应该可以看见我们的测试通过了。

现在是时候创建一个真正的测试了。所有使用Android框架来处理的测试可能都需要一个`instrumentation test`或者使用更复杂的像[Robolectric]库。所以在这些例子中我会不使用框架的任何东西。举个例子，我将测试从`Long`转`String`的扩展函数。

创建一个新的名为`ExtensionTests`的文件，然后增加如下测试：

```kotlin
class ExtensionsTest {
	
	@Test fun testLongToDateString() {
        assertEquals("Oct 19, 2015", 1445275635000L.toDateString())
    }

	@Test fun testDateStringFullFormat() {
        assertEquals("Monday, October 19, 2015",
            1445275635000L.toDateString(DateFormat.FULL))
    }
}
```

这些测试检测`Long`实例是否可以转换成一个`String`。第一个测试默认行为（使用DateFormat.MEDIUM)），而第二个指定一个不同的格式。运行这些测试然后你会看到它们都通过了。我建议你修改它们然后看看它们失败是怎么样的。

如果你在Java中使用过测试，你将会发现这并没有什么太多的不同。我会演示一个简单的例子，我们可以对`ForecastProvider`进行一些测试。我们可以使用`Mockito`库来模拟其它的类然后独立测试provider：

```groovy
dependencies {
    ...
    testCompile "junit:junit:4.12"
    testCompile "org.mockito:mockito-core:1.10.19"
}
```

现在创建了一个`ForecastProviderTest`。我们要去测试`ForecastProvider`，使用`DataSource`来返回结果，看它结果是否为null。所以首先我们需要模拟一个`ForecastDataSource`：

```kotlin
val ds = mock(ForecastDataSource::class.java)
`when`(ds.requestDayForecast(0)).then {
    Forecast(0, 0, "desc", 20, 0, "url")
}
```

如你所见，我们需要在`when`上加反引号。因为`when`在Kotlin中是一个保留关键字，所以如果我们在一些Java代码中使用到它我们需要避免它。现在我们用这个数据源创建了一个provider，然后检测调用那个方法之后的结果是否为null：

```kotlin
val provider = ForecastProvider(listOf(ds))
assertNotNull(provider.requestForecast(0))
```

这是完整的测试函数：

```kotlin
@Test fun testDataSourceReturnsValue() {
    val ds = mock(ForecastDataSource::class.java)
    `when`(ds.requestDayForecast(0)).then {
           Forecast(0, 0, "desc", 20, 0, "url")
    }
    
	val provider = ForecastProvider(listOf(ds))
	assertNotNull(provider.requestForecast(0))
}
```

如果你运行它，你将会看见它会出错。多亏这个测试，我们在自己的代码中发现了某些错误。测试失败是因为`ForecastProvider`在使用之前正在它的`companion object`中初始化。我们可以通过构造函数的方式在`ForecastProvider`中增加一些数据源，这个静态的List就永远不会被使用，所以它应该是使用`lazy`加载：

```kotlin
companion object {
	val DAY_IN_MILLIS = 1000 * 60 * 60 * 24
    val SOURCES by lazy { listOf(ForecastDb(), ForecastServer()) }
}
```

如果你现在再次去运行，你会发现现在会通过所有的测试。
我们也可以测试一些比如当数据源返回null的时候，它会便利下一个数据源来得到结果：

```kotlin
@Test fun emptyDatabaseReturnsServerValue() {
    val db = mock(ForecastDataSource::class.java)
    val server = mock(ForecastDataSource::class.java)
    `when`(server.requestForecastByZipCode(
            any(Long::class.java), any(Long::class.java)))
            .then {
                ForecastList(0, "city", "country", listOf())
    val provider = ForecastProvider(listOf(db, server))
    assertNotNull(provider.requestByZipCode(0, 0))
}
```

如你所见，通过使用参数的默认值这种简单的依赖倒置足够让我们实现一些简单的`unit tests`。对于这个provider还有很多我们可以测试的东西，但是这个例子足够让我们学会使用`unit testing`工具了。

[Robolectric]: http://robolectric.org/