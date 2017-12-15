# If表达式

在Kotlin中一切都是表达式，也就是说一切都返回一个值。如果`if`条件不含有一个exception，那我们可以像我们平时那样使用它：

```kotlin
if(x>0){
	toast("x is greater than 0")
}else if(x==0){ 
	toast("x equals 0")
}else{
	toast("x is smaller than 0")
}
```

我们也可以把结果赋值给一个变量。我们在我们的代码中使用了很多次：

```kotlin
val res = if (x != null && x.size() >= days) x else null
```

这也说明我也不需要像Java那种有一个三元操作符，因为我们可以使用它来简单实现：

```kotlin
val z = if (condition) x else y
```

所以`if`表达式总是返回一个value。如果一个分支返回了Unit，那整个表达式也将返回Unit，它是可以被忽略的，这种情况下它的用法也就跟一般Java中的`if`条件一样了。
