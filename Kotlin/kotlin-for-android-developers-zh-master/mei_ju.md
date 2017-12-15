# 枚举

Kotlin也提供了枚举（`enums`）的实现：

```kotlin
enum class Day {
    SUNDAY, MONDAY, TUESDAY, WEDNESDAY,
    THURSDAY, FRIDAY, SATURDAY
}
```

枚举可以带有参数：

```kotlin
enum class Icon(val res: Int) {
	UP(R.drawable.ic_up),
	SEARCH(R.drawable.ic_search),
	CAST(R.drawable.ic_cast)
}

val searchIconRes = Icon.SEARCH.res
```

枚举可以通过`String`匹配名字来获取，我们也可以获取包含所有枚举的`Array`，所以我们可以遍历它。

```kotlin
val search: Icon = Icon.valueOf("SEARCH")
val iconList: Array<Icon> = Icon.values()
```

而且每一个枚举都有一些函数来获取它的名字、声明的位置：

```kotlin
val searchName: String = Icon.SEARCH.name()
val searchPosition: Int = Icon.SEARCH.ordinal()
```

枚举根据它的顺序实现了 `Comparable`接口，所以可以很方便地把它们进行排序。
