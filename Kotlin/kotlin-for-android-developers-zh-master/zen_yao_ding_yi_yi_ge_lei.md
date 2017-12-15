# 怎么定义一个类

如果你想定义一个类，你只需要使用`class`关键字。
```kotlin
class MainActivity{

}
```

它有一个默认唯一的构造器。我们会在以后的课程中学习在特殊的情况下创建其它额外的构造器，但是请记住大部分情况下你只需要这个默认的构造器。你只需要在类名后面写上它的参数。如果这个类没有任何内容可以省略大括号：

```kotlin
class Person(name: String, surname: String)
```

那么构造函数的函数体在哪呢？你可以写在`init`块中：
```kotlin
class Person(name: String, surname: String) {
    init{
        ...
    }
}
```