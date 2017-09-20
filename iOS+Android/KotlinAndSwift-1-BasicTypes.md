
# Basic

# Kotlin

var 变量
vel 常量

## 基本类型

### 数字
数字的包装类处理类似 Java, 但没有隐式转换(Java 中 int 可以隐式转换成 long)
|Type|Bit width|
|:---|---:|
|Double| 64|
|Float|32|
|Long|64|
|Int|32|
|Short|16|
|Byte|8|

**注意** Kotlin 中的字符不是数字

#### 字面常量

* 十进制：123
  * Long 类型 L 标记
* 十六进制：0x78
* 二进制：0b00000101

**注意**：不支持八进制

浮点常规表示：
* 默认是 double: 123.4e10
* Float 用 f 或 F 标记：123.4f

数字字面值+下划线(1.1 版本支持)
加入下划线可以增加可读性：
``` kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_10100101_10010011_10010010
```
#### 表示方式

数字装箱后不必保留同一性：

``` kotlin
val a: Int = 10000
print(a === a) // 输出“true”
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA === anotherBoxedA) // ！！！输出“false”！！！
```

但保留了同等性：
``` kotlin
val a: Int = 10000
print(a == a) // 输出“true”
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA == anotherBoxedA) // 输出“true”
```

#### 显示转换

不同位宽的类型，无法隐式转换成较宽的类型，如下：
``` kotlin
// 假想的代码，实际上并不能编译：
val a: Int? = 1 // 一个装箱的 Int (java.lang.Integer)
val b: Long? = a // 隐式转换产生一个装箱的 Long (java.lang.Long)
print(a == b) // 惊！这将输出“false”鉴于 Long 的 equals() 检测其他部分也是 Long
```
因此，我们无法将 Byte 赋值给更大位的类型:
``` kotlin
val b: Byte = 1 // OK, 字面值是静态检测的
val i: Int = b // 错误
```
我们必须显示转换成目标类型：
``` kotlin
val i: Int = b.toInt() // OK: 显式拓宽
```
每个数字类型支持如下的转换:
* toByte(): Byte
* toShort(): Short
* toInt(): Int
* toLong(): Long
* toFloat(): Float
* toDouble(): Double
* toChar(): Char

#### 运算
Kotlin支持数字运算的标准集，运算被定义为相应的类成员（但编译器会将函数调用优化为相应的指令）。——可以重载运算符

位运算：没有特殊字符调用，仅能通过中缀方式调用命名函数：
``` kotlin
val i = (1 shl 2) and 0x0000FF00
```
位运算列表（Int,Long支持）
* shl(bits): 有符号左移( << in Java)
* shr(bits)：有符号右移(>> in Java)
* ushr(bits) – 无符号右移 (Java 的 >>>)
* and(bits) – 位与
* or(bits) – 位或
* xor(bits) – 位异或
* inv() – 位非

#### 浮点数比较
本节讨论的浮点数操作如下：

* 相等性检测：a == b 与 a != b
* 比较操作符：a < b、 a > b、 a <= b、 a >= b
* 区间实例以及区间检测：a..b、 x in a..b、 x !in a..b

当其中的操作数 a 与 b 都是静态已知的 Float 或 Double 或者它们对应的可空类型（声明为该类型，或者推断为该类型，或者智能类型转换的结果是该类型），两数字所形成的操作或者区间遵循 IEEE 754 浮点运算标准。

然而，为了支持泛型场景并提供全序支持，当这些操作符并非静态类型为浮点数（例如是 Any、 Comparable<……>、 类型参数）时，这些操作使用为 Float 与 Double 实现的不符合标准的 equals 与 compareTo，这会出现：

* 认为 NaN 与其自身相等
* 认为 NaN 比包括正无穷大（POSITIVE_INFINITY）在内的任何其他元素都大
* 认为 -0.0 小于 0.0（译注：通过 as 方式可在 REPL 中验证，而通过声明变量的方式只能编译运行验证）

> 此处有坑

### 字符
字符用 Char 类型表示。它们不能直接当作数字
``` kotlin
fun check(c: Char) {
    if (c == 1) { // 错误：类型不兼容
        // ……
    }
}
```
字符字面值用单引号括起来: `'1'`。 特殊字符可以用反斜杠转义。 支持这几个转义序列：`\t`、`\b`、`\n`、`\r`、`\'`、`\"`、`\\` 和 `\$`。 编码其他字符要用 Unicode 转义序列语法：`'\uFF00'`。
我们可以显式把字符转换为 Int 数字：
``` kotlin
fun decimalDigitValue(c: Char): Int {
    if (c !in '0'..'9')
        throw IllegalArgumentException("Out of range")
    return c.toInt() - '0'.toInt() // 显式转换为数字
}
```
当需要可空引用时，像数字、字符会被装箱。装箱操作不会保留同一性。

### 布尔

布尔用 Boolean 类型表示，它有两个值：true 和 false。

若需要可空引用布尔会被装箱。

内置的布尔运算有：

* || – 短路逻辑或
* && – 短路逻辑与
* ! - 逻辑非

数组

数组在 Kotlin 中使用 Array 类来表示，它定义了 get 和 set 函数（按照运算符重载约定这会转变为 []）和 size 属性，以及一些其他有用的成员函数：
``` kotlin
class Array<T> private constructor() {
    val size: Int
    operator fun get(index: Int): T
    operator fun set(index: Int, value: T): Unit

    operator fun iterator(): Iterator<T>
    // ……
}
```
我们可以使用库函数 arrayOf() 来创建一个数组并传递元素值给它，这样 arrayOf(1, 2, 3) 创建了 array [1, 2, 3]。 或者，库函数 arrayOfNulls() 可以用于创建一个指定大小、元素都为空的数组。

另一个选项是用接受数组大小和一个函数参数的工厂函数，用作参数的函数能够返回给定索引的每个元素初始值：

// 创建一个 Array<String> 初始化为 ["0", "1", "4", "9", "16"]

``` kotlin
val asc = Array(5, { i -> (i * i).toString() })
```
注意: 与 Java 不同的是，Kotlin 中数组是不型变的（invariant）。这意味着 Kotlin 不让我们把 Array<String> 赋值给 Array<Any>，以防止可能的运行时失败（但是你可以使用 Array<out Any>, 参见类型投影）。

Kotlin 也有无装箱开销的专门的类来表示原生类型数组: ByteArray、 ShortArray、IntArray 等等。这些类和 Array 并没有继承关系，但是它们有同样的方法属性集。它们也都有相应的工厂方法:
``` kotlin
val x: IntArray = intArrayOf(1, 2, 3)
x[0] = x[1] + x[2]
```
### 字符串

字符串用 String 类型表示。字符串是不可变的。 字符串的元素——字符可以使用索引运算符访问: s[i]。 可以用 for 循环迭代字符串:

``` kotlin
for (c in str) {
    println(c)
}
```

转义采用传统的反斜杠方式。参见上面的 字符 查看支持的转义序列。

原生字符串 使用三个引号（"""）分界符括起来，内部没有转义并且可以包含换行和任何其他字符:

``` kotlin
val text = """
    for (c in "foo")
        print(c)
"""
```

你可以通过 trimMargin() 函数去除前导空格：

``` kotlin
val text = """
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()
```

默认 | 用作边界前缀，但你可以选择其他字符并作为参数传入，比如 trimMargin(">")。边界前缀之前的空格会被去除。

#### 字符串模板

字符串可以包含`模板表达式` ，即一些小段代码，会求值并把结果合并到字符串中。 模板表达式以美元符（`$`）开头，由一个简单的名字构成:

``` kotlin
val i = 10
val s = "i = $i" // 求值结果为 "i = 10"
```

或者用花括号括起来的任意表达式:

``` kotlin
val s = "abc"
val str = "$s.length is ${s.length}" // 求值结果为 "abc.length is 3"
```

原生字符串和转义字符串内部都支持模板。 如果你需要在原生字符串中表示字面值 $ 字符（它不支持反斜杠转义），你可以用下列语法：

``` kotlin
val price = """
${'$'}9.99
"""
```


### 定义包

``` kotlin
package my.demo

import java.util.*
```
`目录与包的结构无需匹配：源代码可以在文件系统的任意位置。`

# Swift

var 变量
let 常量

## 字符串插入
通过 `\()` 语法,将括号内的结果插入字符串中

``` swift
let numberOfStopligts :Int = 4
var polulation: Int
polulation = 5422
let townName: String = "Knowhere"
let townDescription = "\(townName.count) has a polulation of \(polulation) and \(numberOfStopligts) stoplights"
print(townDescription)
```

## 数字

### 整数
以 `Int` ，`UInt`声明，存在 Int8,Int16,Int32,Int64。默认 Int 会在被编译器根据设备不同，编译成不同的长度。

支持常规算术运算

#### 溢出操作符

`&+`

``` swift
let x : Int8 = 120 
let y = x &+ 10 // (x + 10) 会是一个 8 位的数字，&+ 符号使之 -126 溢出操作
```

#### 相互转换

不同位宽的整数是无法算术加减，需要转换为同一位宽。

``` swift
let x : Int8 = -120
let y : Int16 = 130
let z = Int16(x) + y
```

建议使用 `Int` 而不是指定位宽的整型。

### 浮点数
默认小数值是 Double 类型

* Double 64
* Float 32

浮点数加减后，实际存储的值可能与算术值有微小差异，判断相等 == 可能出现预期之外的结果

## String

### String 由 Character 组成，且都是 Unicode


