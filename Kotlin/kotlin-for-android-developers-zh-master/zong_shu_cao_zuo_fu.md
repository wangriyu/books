# 总数操作符

#### any

如果至少有一个元素符合给出的判断条件，则返回true。

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
assertTrue(list.any { it % 2 == 0 })
assertFalse(list.any { it > 10 })
```

#### all

如果全部的元素符合给出的判断条件，则返回true。

```kotlin
assertTrue(list.all { it < 10 })
assertFalse(list.all { it % 2 == 0 })
```

#### count

返回符合给出判断条件的元素总数。

```kotlin
assertEquals(3, list.count { it % 2 == 0 })
```

#### fold

在一个初始值的基础上从第一项到最后一项通过一个函数累计所有的元素。

```kotlin
assertEquals(25, list.fold(4) { total, next -> total + next })
```

#### foldRight

与`fold`一样，但是顺序是从最后一项到第一项。

```kotlin
assertEquals(25, list.foldRight(4) { total, next -> total + next })
```

#### forEach

遍历所有元素，并执行给定的操作。

```kotlin
list.forEach { println(it) }
```

#### forEachIndexed

与`forEach`，但是我们同时可以得到元素的index。

```kotlin
list.forEachIndexed { index, value
		-> println("position $index contains a $value") }
```

#### max

返回最大的一项，如果没有则返回null。

```kotlin
assertEquals(6, list.max())
```

#### maxBy

根据给定的函数返回最大的一项，如果没有则返回null。

```kotlin
// The element whose negative is greater
assertEquals(1, list.maxBy { -it })
```

#### min

返回最小的一项，如果没有则返回null。

```kotlin
assertEquals(1, list.min())
```

#### minBy

根据给定的函数返回最小的一项，如果没有则返回null。

```kotlin
// The element whose negative is smaller
assertEquals(6, list.minBy { -it })
```

#### none

如果没有任何元素与给定的函数匹配，则返回true。

```kotlin
// No elements are divisible by 7
assertTrue(list.none { it % 7 == 0 })
```

#### reduce

与`fold`一样，但是没有一个初始值。通过一个函数从第一项到最后一项进行累计。

```kotlin
assertEquals(21, list.reduce { total, next -> total + next })
```

#### reduceRight

与`reduce`一样，但是顺序是从最后一项到第一项。

```kotlin
assertEquals(21, list.reduceRight { total, next -> total + next })
```

#### sumBy

返回所有每一项通过函数转换之后的数据的总和。

```kotlin
assertEquals(3, list.sumBy { it % 2 })
```