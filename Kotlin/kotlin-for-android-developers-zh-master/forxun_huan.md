# For循环

虽然你在使用了collections的函数操作符之后不会再过多地使用for循环，但是for循环再一些情况下仍然是很有用的。提供一个迭代器它可以作用在任何东西上面：

```kotlin
for (item in collection) {
	print(item)
}
```

如果你需要更多使用index的典型的迭代，我们也可以使用`ranges`（反正它通常是更加智能的解决方案）：

```kotlin
for (index in 0..viewGroup.getChildCount() - 1) {
    val view = viewGroup.getChildAt(index)
    view.visibility = View.VISIBLE
}
```

在我们迭代一个array或者list，一系列的index可以用来获取到指定的对象，所以上面的方式不是必要的：

```kotlin
for (i in array.indices)
	print(array[i])
```