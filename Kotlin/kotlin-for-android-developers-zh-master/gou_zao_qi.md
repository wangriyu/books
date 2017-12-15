# 构造器

所有构造函数默认都是`public`的，它们类是可见的，可以被其它地方使用。我们也可以使用这个语法来把构造函数修改为`private`：

```kotlin
class C private constructor(a: Int) { ... }
```