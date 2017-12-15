# 可null性和Java库

好了，前面的章节解释了使用Kotlin代码完美地工作。但是与普通的Java库和Android SDK会发生什么呢？在Java中，所有对象可以被定义为null。所以我们不得不处理大量潜在的在现实中不可能是null的null变量。这意味着我们的代码最后可能会有几百个`!!`操作符，这绝对不是一个好的主意。

当我们去处理Android SDK时，你可能看见所有Java方法的参数被标记为单个的`!`。比如，Java中在一些获取对象的方法在Kotlin中显示返回`Any!`。这表示让开发者自己决定是否这个变量是否可null。

很幸运，新版本的Android开始使用`@Nullable`和`@NonNull`注解来辨别参数是否可以是null或者否个函数是否可以返回null。当我们怀疑时，我们可以进入源码去检查是否会接收到一个null对象。我的猜想是在以后，编译器能够读取这些注解，然后强制（或者至少是建议）一个更好的方法。

现在开始，当一个Jetbrains的`@Nullable`注解（这个与Android的注解不同）被注解在一个非null的变量时，就会获得一个警告。相对的没有发生在`@NotNull`注解上。

所以我们来举个例子，如果我们创建了一个Java的测试类：

```java
import org.jetbrains.annotations.Nullable;
public class NullTest {

	@Nullable
	public Object getObject(){
		return "";
	}
}
```

然后在Kotlin中使用：

```kotlin
val test = NullTest()
val myObject: Any = test.getObject()
```

我们会发现，在`getObject`函数上会显示一个警告。但是这只是从现在才开始的编译器检查，并且它还不认识Android的注解，所以我们可能不得不花更多的时间来等待一个更智能的方式。不管怎么样，使用源码注解的方式和一些Androd SDK的知识，我们也很难犯错误。

比如重写`Activity`的`onCraete`函数，我们可以决定是否让`savedInstanceState`可null：

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
}

override fun onCreate(savedInstanceState: Bundle) {
}
```

这两种方法都会被编译，但是第二种是错误的，因为一个Activity很可能接收到一个null的bundle。只要小心一点点就足够了。当你有疑问时，你可以就用可null的对象然后处理掉用可能的null。记住，如果你使用了 `!!`，可能是因为你确信对象不可能为null，如果是这样，请定义为非null。

这个灵活性在Java库中真的很有必要，而且随着编译器的进化，我们将可能看到更好的交互（可能是基于注解的），但是现在来说这个机制已经足够灵活了。