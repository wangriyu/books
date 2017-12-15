# 操作符表

这里你可以看见一系列包括`操作符`和`对应方法`的表。对应方法必须在指定的类中通过各种可能性被实现。

__一元操作符__

| 操作符 | 函数|
|----|
| +a | a.unaryPlus() |
| -a | a.unaryMinus() |
| !a | a.not() |
| a++ | a.inc() |
| a-- | a.dec() |
<br/>
__二元操作符__

| 操作符 | 函数|
|----|
| a + b | a.plus(b) |
| a - b | a.minus(b) |
| a * b | a.times(b) |
| a / b | a.div(b) |
| a % b | a.mod(b) |
| a..b | a.rangeTo(b) |
| a in b | a.contains(b) |
| a !in b | !a.contains(b) |
| a += b | a.plusAssign(b) |
| a -= b | a.minusAssign(b) |
| a *= b | a.timesAssign(b) |
| a /= b | a.divAssign(b) |
| a %= b | a.modAssign(b) |
<br/>
__数组操作符__

| 操作符 | 函数|
|----|
| a[i] | a.get(i) |
| a[i, j] | a.get(i, j) |
| a[i\_1, ..., i\_n] | a.get(i\_1, ..., i\_n) |
| a[i] = b | a.set(i, b) |
| a[i, j] = b | a.set(i, j, b) |
| a[i\_1, ..., i\_n] = b | a.set(i\_1, ..., i\_n, b) |
<br/>
__等于操作符__

| 操作符 | 函数|
|----|
| a == b | a?.equals(b) ?: b === null |
| a != b | !(a?.equals(b) ?: b === null) |

相等操作符有一点不同，为了达到正确合适的相等检查做了更复杂的转换，因为要得到一个确切的函数结构比较，不仅仅是指定的名称。方法必须要如下准确地被实现：

```kotlin
operator fun equals(other: Any?): Boolean
```
操作符`===`和`!==`用来做身份检查（它们分别是Java中的`==`和`!=`），并且它们不能被重载。

<br/>
__函数调用__

| 方法 | 调用 |
|----|
| a(i) | a.invoke(i) |
| a(i, j) | a.invoke(i, j) |
| a(i\_1, ..., i\_n)| a.invoke(i\_1, ..., i\_n) |