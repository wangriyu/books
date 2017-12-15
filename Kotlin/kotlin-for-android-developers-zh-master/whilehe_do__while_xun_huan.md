# While和do/while循环

你也可以使用`while`循环，尽管它们两个都不是特别常用的。它们通常可以更简单、视觉上更容易理解的方式去解决一个问题，两个例子：

```kotlin
while(x > 0){ 
	x--
}

do{
	val y = retrieveData()
} while (y != null) // y在这里是可见的!
```