# Kotlin中的null安全

如果你正在使用Java 7工作的话，null安全是Kotlin中最令人感兴趣的特性之一了。但是就如你在本书中看到的，它好像不存在一样，一直到上一章我们几乎都不需要去担心它。

通过[我们自己创造的亿万美金的错误]对null的思考，我们有时候的确需要去定义一个变量包不包含一个值。在Java中尽管注解和IDE在这方面帮了我们很多，但是我们仍然可以这么做：

```kotlin
Forecast forecast = null;
forecast.toString();
```

这个代码可以被完美地编译（你可能会从IDE上得到一个警告），然后正常地执行，但是显然它会抛一个`NullPointerException`。这个相当不安全的。而且按照我们的想法，我们应该去控制一切，随着代码的增长，我们会慢慢对某些null的控制。所以最终会得到很多的`NullPointerException`或者丢失很多null检查（可能两者混合）。

[我们自己创造的亿万美金的错误]: https://en.wikipedia.org/wiki/Tony_Hoare
