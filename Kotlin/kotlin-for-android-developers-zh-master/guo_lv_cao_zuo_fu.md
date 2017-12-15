# 过滤操作符

#### drop

返回包含去掉前n个元素的所有元素的列表。

```kotlin
assertEquals(listOf(5, 6), list.drop(4))
```

#### dropWhile

返回根据给定函数从第一项开始去掉指定元素的列表。

```kotin
assertEquals(listOf(3, 4, 5, 6), list.dropWhile { it < 3 })
```

#### dropLastWhile

返回根据给定函数从最后一项开始去掉指定元素的列表。

```kotlin
assertEquals(listOf(1, 2, 3, 4), list.dropLastWhile { it > 4 })
```

#### filter

过滤所有符合给定函数条件的元素。

```kotin
assertEquals(listOf(2, 4, 6), list .ilter { it % 2 == 0 })
```

#### filterNot

过滤所有不符合给定函数条件的元素。

```kotin
assertEquals(listOf(1, 3, 5), list.filterNot { it % 2 == 0 })
```

#### filterNotNull

过滤所有元素中不是null的元素。

```kotlin
assertEquals(listOf(1, 2, 3, 4), listWithNull.filterNotNull())
```

#### slice

过滤一个list中指定index的元素。

```kotlin
assertEquals(listOf(2, 4, 5), list.slice(listOf(1, 3, 4)))
```

#### take

返回从第一个开始的n个元素。

```kotlin
assertEquals(listOf(1, 2), list.take(2))
```

#### takeLast

返回从最后一个开始的n个元素

```kotlin
assertEquals(listOf(5, 6), list.takeLast(2))
```

#### takeWhile

返回从第一个开始符合给定函数条件的元素。

```kotlin
assertEquals(listOf(1, 2), list.takeWhile { it < 3 })
```