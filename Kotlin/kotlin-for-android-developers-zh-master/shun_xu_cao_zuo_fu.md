# 顺序操作符

#### reverse

返回一个与指定list相反顺序的list。

```kotlin
val unsortedList = listOf(3, 2, 7, 5)
assertEquals(listOf(5, 7, 2, 3), unsortedList.reverse())
```

#### sort

返回一个自然排序后的list。

```kotlin
assertEquals(listOf(2, 3, 5, 7), unsortedList.sort())
```

#### sortBy

返回一个根据指定函数排序后的list。

```kotlin
assertEquals(listOf(3, 7, 2, 5), unsortedList.sortBy { it % 3 })
```

#### sortDescending

返回一个降序排序后的List。

```kotlin
assertEquals(listOf(7, 5, 3, 2), unsortedList.sortDescending())
```

#### sortDescendingBy

返回一个根据指定函数降序排序后的list。

```kotlin
assertEquals(listOf(2, 5, 7, 3), unsortedList.sortDescendingBy { it % 3 })
```