# Ranges

很难解释`control flow`，如果不去讲讲`ranges`的话。但是它们的范围要宽得多。`Range`表达式使用一个`..`操作符，它是被定义实现了一个`RangTo`方法。

`Ranges`帮助我们使用很多富有创造性的方式去简化我们的代码。比如我们可以把它：

```kotlin
if(i >= 0 && i <= 10) 
	println(i)
```

转化成：

```kotlin
if (i in 0..10)
    println(i)
```

`Range`被定义为可以被比较的任意类型，但是对于数字类型，比较器会通过转换它为简单的类似Java代码来避免额外开销的方式来优化它。数字类型的`ranges`也可以被迭代，编译器会转换它们为与Java中使用index的for循环的相同字节码的方式来进行优化：

```kotlin
for (i in 0..10)
    println(i)
```

`Ranges`默认会自增长，所以如果像以下的代码：

```kotlin
for (i in 10..0)
    println(i)
```

它就不会做任何事情。但是你可以使用`downTo`函数：

```kotlin
for(i in 10 downTo 0)
	println(i)
```

我们可以在`range`中使用`step`来定义一个从1到一个值的不同的空隙：

```kotlin
for (i in 1..4 step 2) println(i)

for (i in 4 downTo 1 step 2) println(i)
```

如果你想去创建一个open range（不包含最后一项，译者注：类似数学中的开区间），你可以使用`until`函数：

```kotlin
for (i in 0 until 4) println(i)
```

这一行会打印从0到3，但是会跳过最后一个值。这也就是说`0 until 4 == 0..3`。在一个list中迭代时，使用`(i in 0 until list.size)`比`(i in 0..list.size - 1)`更加容易理解。

就如之前所提到的，使用`ranges`确实有富有创造性的方式。比如，一个简单的方式去从一个`ViewGroup`中得到一个Views列表可以这么做：

```kotlin
val views = (0..viewGroup.childCount - 1).map { viewGroup.getChildAt(it) }
```

混合使用`ranges`和`函数操作符`可以避免我们使用明确地循环去迭代一个集合，还有明确地去创建一个我们用来添加views的list。所有的事情都在一行代码中做好了。

如果你想知道更多`ranges`的实现方式和更多的范例和游泳的信息，你可以进入[Kotlin reference]

[Kotlin reference]:  https://kotlinlang.org/docs/reference/ranges.html