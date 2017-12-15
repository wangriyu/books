# ManagedSqliteOpenHelper

Anko提供了很多强大的SqliteOpenHelper来可以大量简化代码。当我们使用一个一般的`SqliteOpenHelper`，我们需要去调用`getReadableDatabase()`或者`getWritableDatabase()`，然后我们可以执行我们的搜索并拿到结果。在这之后，我们不能忘记调用`close()`。使用`ManagedSqliteOpenHelper`我们只需要：

```kotlin
forecastDbHelper.use {
	...
}
```

在lambda里面，我们可以直接使用`SqliteDatabase`中的函数。它是怎么工作的？阅读Anko函数的实现方式真是一件有趣的事情，你可以从这里学到Kotlin的很多知识：

```kotlin
public fun <T> use(f: SQLiteDatabase.() -> T): T {
    try {
	    return openDatabase().f()
	} finally {
	    closeDatabase()
    }
}
```

首先，`use`接收一个`SQLiteDatabase`的扩展函数。这表示，我们可以使用`this`在大括号中，并且处于`SQLiteDatabase`对象中。这个函数扩展可以返回一个值，所以我们可以像这么做：

```kotin
val result = forecastDbHelper.use {
    val queriedObject = ...
    queriedObject
}
```

要记住，在一个函数中，最后一行表示返回值。因为T没有任何的限制，所以我们可以返回任何对象。甚至如果我们不想返回任何值就使用`Unit`。

由于使用了`try-finall`，`use`方法会确保不管在数据库操作执行成功还是失败都会去关闭数据库。

而且，在`sqliteDatabase`中还有很多有用的扩展函数，我们会在之后使用到他们。但是现在让我们先定义我们的表和实现`SqliteOopenHelper`。