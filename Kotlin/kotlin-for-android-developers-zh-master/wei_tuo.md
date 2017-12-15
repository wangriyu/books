# 委托

[委托模式]是一个很有用的模式，它可以用来从类中抽取出主要负责的部分。委托模式是Kotlin原生支持的，所以它避免了我们需要去调用委托对象。委托者只需要指定实现的接口的实例。

在我们前面的例子中，我们可以通过构造函数指定动物怎么飞行，而不是实现它。比如，一个使用翅膀飞行的动物可以用这种方式指定：

```kotlin
interface CanFly {
	fun fly()
}

class Bird(f: CanFly) : CanFly by f
```

我们可以使用接口来指示鸟可以飞行，但是鸟的飞行方式被定义在一个委托中，这个委托定义在构造函数中，所以我们可以针对不同的鸟使用不同的飞行方式。动物使用翅膀飞行的方式被定义在另一个类中：

```kotlin
class AnimalWithWings : CanFly {
    val wings: Wings = Wings()
    override fun fly() = wings.move()
}
```

动物扇动翅膀来飞行。所以我们可以创建一个鸟，它使用翅膀飞行：

```kotlin
val birdWithWings = Bird(AnimalWithWings())
birdWithWings.fly()
```

但是现在翅膀可以被别的不是鸟类的动物使用。如果我们假设蝙蝠使用翅膀，我们可以直接指定委托来实例化对象：

```kotlin
class Bat : CanFly by AnimalWithWings()
...
val bat = Bat()
bat.fly()
```

[委托模式]: https://en.wikipedia.org/wiki/Delegation_pattern