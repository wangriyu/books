# 接口

Kotlin中的接口比Java 7中要强大得多。如果你使用Java 8，它们非常相似。在Kotlin中，我们可以像Java中那样使用接口。想象我们有一些动物，它们的其中一些可以飞行。这个是我们针对飞行动物的接口：

```kotlin
interface FlyingAnimal {
	fun fly()
}
```

鸟和蝙蝠都可以通过扇动翅膀的方式飞行。所以我们为它们创建两个类：

```kotlin
class Bird : FlyingAnimal {
    val wings: Wings = Wings()
    override fun fly() = wings.move()
}

class Bat : FlyingAnimal {
    val wings: Wings = Wings()
    override fun fly() = wings.move()
}
```

当两个类继承自一个接口，非常典型的是它们两者共享相同的实现。但是Java 7中的接口只能定义行为，但是不能去实现它。

Kotlin接口在某一方面它可以实现函数。它们与类唯一的不同之处是它们是无状态（stateless）的，所以属性需要子类去重写。类需要去负责保存接口属性的状态。

我们可以让接口实现`fly`函数：

```kotlin
interface FlyingAnimal {
    val wings: Wings
    fun fly() = wings.move()
}
```

就像提到的那样，类需要去重写属性：

```kotlin
class Bird : FlyingAnimal {
	override val wings: Wings = Wings()
}

class Bat : FlyingAnimal {
	override val wings: Wings = Wings()
}
```

现在鸟和蝙蝠都可以飞行了：

```kotlin
val bird = Bird()
val bat = Bat()

bird.fly()
bat.fly()
```