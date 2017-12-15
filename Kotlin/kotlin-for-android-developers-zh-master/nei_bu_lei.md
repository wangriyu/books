# 内部类

在Java中，我们可以在类的里面再定义类。如果它是一个通常的类，它不能去访问外部类的成员（就如Java中的static）：

```kotlin
class Outer {
  private val bar: Int = 1
	class Nested {
		fun foo() = 2
	}
}

val demo = Outer.Nested().foo() // == 2
```

如果需要去访问外部类的成员，我们需要用`inner`声明这个类：

```kotlin
class Outer {
	private val bar: Int = 1
	inner class Inner{
		fun foo() = bar
	}
}

val demo = Outer().Inner().foo() // == 1
```
