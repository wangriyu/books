# 基本类型

当然，像integer，float或者boolean等类型仍然存在，但是它们全部都会作为对象存在的。基本类型的名字和它们工作方式都是与Java非常相似的，但是有一些不同之处你可能需要考虑到：

- 数字类型中不会自动转型。举个例子，你不能给`Double`变量分配一个`Int`。必须要做一个明确的类型转换，可以使用众多的函数之一：

```kotlin
val i:Int=7
val d: Double = i.toDouble()
```

- 字符（Char）不能直接作为一个数字来处理。在需要时我们需要把他们转换为一个数字：

```kotlin
val c:Char='c'
val i: Int = c.toInt()
```

- 位运算也有一点不同。在Android中，我们经常在`flags`中使用“或”，所以我使用"and"和"or来举例：

```java
// Java
int bitwiseOr = FLAG1 | FLAG2;
int bitwiseAnd = FLAG1 & FLAG2;
```

```kotlin
// Kotlin
val bitwiseOr = FLAG1 or FLAG2
val bitwiseAnd = FLAG1 and FLAG2
```

>还有很多其他的位操作符，比如`sh1`, `shs`, `ushr`, `xor`或`inv`。当我们需要的时候，可以在[Kotlin官网]查看。

- 字面上可以写明具体的类型。这个不是必须的，但是一个通用的Kotlin实践时省略变量的类型（我们马上就能看到），所以我们可以让编译器自己去推断出具体的类型。

```kotlin
val i = 12 // An Int
val iHex = 0x0f // 一个十六进制的Int类型
val l = 3L // A Long
val d = 3.5 // A Double
val f = 3.5F // A Float
```

- 一个String可以像数组那样访问，并且被迭代：

```kotlin
val s = "Example"
val c = s[2] // 这是一个字符'a'
// 迭代String
val s = "Example"
for(c in s){
    print(c)
}
```

[kotlin官网]: http://kotlinlang.org/docs/reference/basic-types.html#operations