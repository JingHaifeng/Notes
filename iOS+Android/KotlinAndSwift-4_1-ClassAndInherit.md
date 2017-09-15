## 类

### 声明
Kotlin 中使用关键字 class 声明类

``` kotlin
class Invoice {
}
```

类声明由类名、类头（指定其类型参数、主构造函数等）和由大括号包围的类体构成。类头和类体都是可选的； 如果一个类没有类体，可以省略花括号。

``` kotlin
class Empty
```

### 构造函数
在 Kotlin 中的一个类可以有一个主构造函数和一个或多个次构造函数。主构造函数是类头的一部分：它跟在类名（和可选的类型参数）后。

``` koltin
class Person constructor(firstName: String) {
}
```

如果主构造函数**没有任何**`注解`或者`可见性修饰符`，可以省略这个 constructor 关键字。

``` kotlin
class Person(firstName: String) {
}
```

主构造函数不能包含任何的代码。初始化的代码可以放到以 `init` 关键字作为前缀的初始化块（initializer blocks）中：

``` kotlin
class Customer(name: String) {
    init {
        logger.info("Customer initialized with value ${name}")
    }
}
```

**注意**：主构造的参数可以在初始化块中使用。它们也可以在类体内声明的属性初始化器中使用：

``` kotlin
class Customer(name: String) {
    val customerKey = name.toUpperCase()
}
```
事实上，声明属性以及从主构造函数初始化属性，Kotlin 有简洁的语法：

``` koltin
class Person(val firstName: String, val lastName: String, var age: Int) {
    // ……
}
```

与普通属性一样，主构造函数中声明的属性可以是可变的（var）或只读的（val）。

如果构造函数有注解或可见性修饰符，这个 constructor 关键字是必需的，并且这些修饰符在它前面：

``` kotlin
class Customer public @Inject constructor(name: String) { …… }
```

#### 次构造函数
类也可以声明前缀有 constructor的次构造函数：

``` kotlin
class Person {
    constructor(parent: Person) {
        parent.children.add(this)
    }
}
```

如果类有一个主构造函数，每个次构造函数需要委托给主构造函数， 可以直接委托或者通过别的次构造函数间接委托。委托到同一个类的另一个构造函数用 this 关键字即可：

``` kotlin
class Person(val name: String) {
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}
```

如果一个非抽象类没有声明任何（主或次）构造函数，它会有一个生成的不带参数的主构造函数。构造函数的可见性是 public。如果你不希望你的类有一个公有构造函数，你需要声明一个带有非默认可见性的空的主构造函数：

``` kotlin
class DontCreateMe private constructor () {
}
```

**注意**：在 JVM 上，如果主构造函数的所有的参数都有默认值，编译器会生成 一个额外的无参构造函数，它将使用默认值。这使得 Kotlin 更易于使用像 Jackson 或者 JPA 这样的通过无参构造函数创建类的实例的库。

``` kotlin
class Customer(val customerName: String = "")
```