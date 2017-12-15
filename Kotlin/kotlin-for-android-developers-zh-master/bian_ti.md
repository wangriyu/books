# 变体

这是真的是最难理解的部分之一。在Java中，当我们使用泛型的时候会出现问题。逻辑告诉我们`List<String>`应该可以转型为`List<Object>`，因为它有更弱的限制。但是我们来看下这个例子：

```kotlin
List<String> strList = new ArrayList<>();
List<Object> objList = strList;
objList.add(5);
String str = objList.get(0);
```

如果Java编译器允许我们这么做，我们可以增加一个`Integer`到`Object` List，但是它明显会在某一时刻奔溃。这就是为什么语言中增加了通配符。通配符可以在限制这个问题中可以增加灵活性。

如果我们增加了`? extends Object`，我们使用了协变（`covariance`），它表示我们可以处理任何使用了类型，比Object更严格的对象，但是我们只有使用`get`操作时是安全的。如果我们想去拷贝一个`Strings`集合到`Objects`集合中，我们应该是允许的，对吧？然后，如果我们这样：

```kotlin
List<String> strList = ...;
List<Object> objList = ...;
objList.addAll(strList);
```

这样是可以的，因为定义在`Collection`接口中的`addAll()`是这样的：

```java
List<String>
interface Collection<E> ... {
	void addAll(Collection<? extends E> items);
}
```

否则，没有通配符，我们不会允许在这个方法中使用`String` List。相反地，当然会失败。我们不能使用`addAll()`来增加一个`Objects` List到`Strings` List中。因为我们只是用那个方法从`collection`中获取元素，这是一个完美的协变（`covariance`）的例子。

另一方面，我们可以在对立面上发现逆变（`contravariance`）。按照集合的例子，如果我们想把传过来的参数增加到集合中去，我们可以增加更加限制的类型到泛型集合中。比如，我们可以增加`Strings`到`Object`List：

```java
void copyStrings(Collection<? super String> to, Collection<String> from) {
    to.addAll(from);
}
```

增加`Strings`到另一个集合中唯一的限制就是那个集合接收`Strings`或者父类。

但是通配符都有它的限制。通配符定义了使用场景变体（`use-site variance`），这意味着当我们使用它的时候需要声明它。这表示每次我们声明一个泛型变量时都会增加模版代码。

让我们看一个例子。使用我们之前相似的类：

```java
class TypedClass<T> {
    public T doSomething(){
	    ...
    }
}
```

这些代码不会被编译：

```java
TypedClass<String> t1 = new TypedClass<>();
TypedClass<Object> t2 = t1;
```

尽管它的确没有意义，因为我们仍然保持了类中的所有的方法并且没有任何损坏。我们需要指定的类型可以有一个更加灵活的定义。

```kotlin
TypedClass<String> t1 = new TypedClass<>();
TypedClass<? extends String> t2 = t1;
```

这会让代码更加难以理解，而且增加了一些额外的模版代码。

另一方面，Kotlin通过内部声明变体（`declaration-site variance`）可以使用更加容易的方式来处理。这表示当我们定义一个类或者接口的时候我们可以处理弱限制的场景，我们可以在其它地方直接使用它。

所以让我们看看它在Kotlin中是怎么工作的。相比冗长的通配符，Kotlin仅仅使用`out`来针对协变（`covariance`）和使用`in`来针对逆变（`contravariance`）。在这个例子中，当我们类产生的对象可以被保存到弱限制的变量中，我们使用协变。我们可以直接在类中定义声明｛

```kotlin
class TypedClass<out T>() {
    fun doSomething(): T {
	    ...
	}
}
```

这就是所有我们需要的。现在，在Java中不能编译的代码在Kotlin中可以完美运行：

```kotlin
val t1 = TypedClass<String>()
val t2: TypedClass<Any> = t1
```

如果你已经使用了这些概念，我确信你可以很简单地在Kotlin使用`in`和`out`。否则，你也只是需要一些联系和概念上的理解。