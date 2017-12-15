# 生产操作符

#### merge

把两个集合合并成一个新的，相同index的元素通过给定的函数进行合并成新的元素作为新的集合的一个元素，返回这个新的集合。新的集合的大小由最小的那个集合大小决定。

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
val listRepeated = listOf(2, 2, 3, 4, 5, 5, 6)
assertEquals(listOf(3, 4, 6, 8, 10, 11), list.merge(listRepeated) { it1, it2 -> it1 + it2 })
```

#### partition

把一个给定的集合分割成两个，第一个集合是由原集合每一项元素匹配给定函数条件返回`true`的元素组成，第二个集合是由原集合每一项元素匹配给定函数条件返回`false`的元素组成。

```kotlin
assertEquals(
	Pair(listOf(2, 4, 6), listOf(1, 3, 5)), 
	list.partition { it % 2 == 0 }
)
```

#### plus

返回一个包含原集合和给定集合中所有元素的集合，因为函数的名字原因，我们可以使用`+`操作符。

```kotlin
assertEquals(
	listOf(1, 2, 3, 4, 5, 6, 7, 8), 
	list + listOf(7, 8)
)
```

#### zip

返回由`pair`组成的List，每个`pair`由两个集合中相同index的元素组成。这个返回的List的大小由最小的那个集合决定。

```kotin
assertEquals(
	listOf(Pair(1, 7), Pair(2, 8)), 
	list.zip(listOf(7, 8))
)
```

#### unzip

从包含pair的List中生成包含List的Pair。

```kotlin
assertEquals(
	Pair(listOf(5, 6), listOf(7, 8)), 
	listOf(Pair(5, 7), Pair(6, 8)).unzip()
)
```