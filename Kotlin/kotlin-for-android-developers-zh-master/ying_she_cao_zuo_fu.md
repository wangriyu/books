# 映射操作符

#### flatMap

遍历所有的元素，为每一个创建一个集合，最后把所有的集合放在一个集合中。

```kotlin
assertEquals(listOf(1, 2, 2, 3, 3, 4, 4, 5, 5, 6, 6, 7), 
list.flatMap { listOf(it, it + 1) })
```

#### groupBy

返回一个根据给定函数分组后的map。

```kotlin
assertEquals(mapOf("odd" to listOf(1, 3, 5), "even" to listOf(2, 4, 6)), list.groupBy { if (it % 2 == 0) "even" else "odd" })
```

#### map

返回一个每一个元素根据给定的函数转换所组成的List。

```kotlin
assertEquals(listOf(2, 4, 6, 8, 10, 12), list.map { it * 2 })
```

#### mapIndexed

返回一个每一个元素根据给定的包含元素index的函数转换所组成的List。

```kotlin
assertEquals(listOf (0, 2, 6, 12, 20, 30), list.mapIndexed { index, it -> index * it })
```

#### mapNotNull

返回一个每一个非null元素根据给定的函数转换所组成的List。

```kotlin
assertEquals(listOf(2, 4, 6, 8), listWithNull.mapNotNull { it * 2 })
```