# When表达式

`when`表达式与Java中的`switch/case`类似，但是要强大得多。这个表达式会去试图匹配所有可能的分支直到找到满意的一项。然后它会运行右边的表达式。与Java的`switch/case`不同之处是参数可以是任何类型，并且分支也可以是一个条件。

对于默认的选项，我们可以增加一个`else`分支，它会在前面没有任何条件匹配时再执行。条件匹配成功后执行的代码也可以是代码块：

```kotlin
when (x){
	1 -> print("x == 1") 
	2 -> print("x == 2") 
	else -> {
		print("I'm a block")
		print("x is neither 1 nor 2")
    }
}
```

因为它是一个表达式，它也可以返回一个值。我们需要考虑什么时候作为一个表达式使用，它必须要覆盖所有分支的可能性或者实现`else`分支。否则它不会被编译成功：

```kotlin
val result = when (x) {
    0, 1 -> "binary"
	else -> "error"
}
```

如你所见，条件可以是一系列被逗号分割的值。但是它可以更多的匹配方式。比如，我们可以检测参数类型并进行判断：

```kotlin
when(view) {
    is TextView -> view.setText("I'm a TextView")
    is EditText -> toast("EditText value: ${view.getText()}")
    is ViewGroup -> toast("Number of children: ${view.getChildCount()} ")
    else -> view.visibility = View.GONE
}
```

再条件右边的代码中，参数会被自动转型，所以你不需要去明确地做类型转换。

它还让检测参数否在一个数组范围甚至是集合范围成为可能（我会在这章节的后面讲这个）：

```kotlin
val cost = when(x) {
	in 1..10 -> "cheap"
	in 10..100 -> "regular"
	in 100..1000 -> "expensive"
	in specialValues -> "special value!"
	else -> "not rated"
}
```

或者你甚至可以从对参数做需要的几乎疯狂的检查摆脱出来。它可以使用简单的`if/else`链替代：

```kotlin
valres=when{
	x in 1..10 -> "cheap"
	s.contains("hello") -> "it's a welcome!"
	v is ViewGroup -> "child count: ${v.getChildCount()}"
	else -> ""
}
```