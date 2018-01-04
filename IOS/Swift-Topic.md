# Swift Topic

## Optional

### An Optional is just an enum

In other words ...
``` swift
enum Optional<T> { // The <T> is a generic like as in Array<T>
    case none
    case some(T)
}
```

Some example: wrapped
``` swift
let x: String? = nil
// ... is ...
let x = Optional<String>.none

let x: String? = "hello"
// ... is ...
let x = Optional<String>.some("hello")
```

Some example: unwrapped
``` swift
let y = x!
// ... is ...
switch x {
    case some(let value): y = value
    case none:// raise an exception
}
```

Some example: if let
``` swift
let x: String? = ...
if let y = x {
    // do some thing with y
}
// ... is ...
switch x {
    case .some(let y):
        // do some thing with y
    case .none:
        break
}
```

### Optionals can be "chained"

For example, hashValue is a var in String.

What is we wanted to `get the hashValue  from` an Optional String?

And what if that Optional String was, itself, `the text of an Optional UILabel`?

``` swift
var display: UILabel? 
if let temp1 = display {
    if let temp2 = temp1.text {
        let x = temp2.hashValue
        // ...
    }
}
// ... with Optional chaining using ? instead of ! to unwrap, this becomes ...
if let x = display?.text?.hashValue { ... } // x is an Int
   let x = display?.text?.hashValue { ... } // x is an Int?
```

### There is also an Optional "defaulting" operator `??`
What if we want to put a String into a UILabel, but if it's nil, put " "

``` swift
let s: String? = ... // might be nil
if s!= nil {
    display.text = s
} else {
    display.text = " "
}
// can be expressed much more simply this way ...
display.text = s ?? " "
```
## Tuples
It is nothing more than a grouping of values.

You can use it anywhere you can use a type.

``` swift
let x: (String, Int, Double) = ("hello", 5, 0.85) // the type of x is a tuple
let (word, number, value) = x // this names the tuple elements when accessing the tuple
print(word)
print(number)
print(value)
// ... or the tuple elements can be named when the tuple is declared (this is strongly preferred)
let x: (w: String, i: Int, v: Double) = ("hello", 5, 0.85)
print(x.w)
print(x.i)
print(x.v)
```

### Tuples as return values
You can use tuples to return multiple values from a fuction or method ...

``` swift
func getSize() -> (weight: Double, height: Double) {
    return (250, 80)
}

let x = getSize()
print("weight is \(x.weight)") // weight is 250
// or
print("height is \(getSize().height)") // height is 80
```

## Range
A Range in Swift is just two end points.

A Range can represent things like a selection in some text or a portion of an Array.

Range is generic (e.g. Range<T>), but T is restricted (e.g. comparable).

This is sort of a pseudo-representation of Range ...

``` swift
struct Range<T> {
    var startIndex: T
    var endIndex: T
}
```

So, for example, a Range<Int> would be good for a range specifying a slice of an Array.

There are other, more capable, Ranges like CountableRange.

A `CountableRange` contains consecutive values which can be iterated over or indexed into.

There is special syntax for creating a Range.

Either `..<` (exclusive of the upper bound) or `...` (inclusive of both bounds)

``` swift
let array = ["a","b","c","d"]
let a = array[2...3] // a will be a slice of the array containing ["c","d"]
let b = array[2..<3] // b will be a slice of the array containing ["c"]
let c = array[6...8] // runtime crash (array index out of bounds)
let d = array[4...1] // runtime crash (lower bound must be smaller than upper bound)
```

A String subrange is **not** Range<Int> (it's Range<String.Index>)

``` swift
let e = "hello"[2..<4] // this != "ll",in fact, it won't even compile
let f = "hello"[start..<end] // this is possible; we'll explain start and end a bit later
```

If the type of the upper/lower bound is an Int, `..<` makes a ContableRange.
(Actually, it depends on whether the upper/lower bound is "strideable by Int" to be precise.)

CountableRange is enumeratable with `for in`.

For example, this is how you do a C-like `for (i = 0; i < 20; i++)` loop ...

``` swift
for i in 0..<20 {

}
```

How about something like `for (i = 0.5; i <= 15.25; i += 0.3)`?

Floating point numbers don't stride by Int, they stride by a floating point value.

So `0.5...15.25` is just a Range, not a CountableRange (which is needed for for in).

Luckily, there's a global function that will create a CountableRange from floating point values!

``` swift
for i in stride(from: 0.5, through: 15.25, by: 0.3) {

}
```

The return type of stride is CountableRange (actually `ClosedCountableRange` in this case).

## Data Structures in Swift

### Classes, Structures and Enumerations

These are the 3 of the 4 fundational building blocks of data structures in Swift

### Similarities

Declaration syntax ...

``` swift
class ViewController: ... { // class can specify a supper class

}

struct CalculatorBrain {

}

enum Op {

}
```

Properties and Functions ...

>枚举把它的数据保存在关联值中，所以不能存储任何属性，但它可以有计算属性和函数。

``` swift
func doit (argx argi: Type) -> ReturnValue {

}

var storedProperty = <initial value> (not enum)

var computedProperty: Type {
    get {}
    set {}
}
```

Initializes (again, not enum) ...

``` swift
init (arg1x arg1i: Type, arg2x arg2i: Type, ...) {

}
```

### Differences

Inheritance (class only)

Value type (struct, enum) vs. Reference type(class)

## Value vs. Reference

### Value (`sturct` and `enum`)

Copied when passed as an argument to a function

Copied when assigned to a different var

Immutable if assigned to a variable with `let` (function parameters are `let`)

You must note any `func` that can mutate a struct/enum with the keyword `mutating`

### Reference (`class`)

Stored in the heap and reference couted (automatically)

Constant pointers to a class (`let`) still can mutate by calling methods and changing properties

When passed as an argument, does not make a copy(just passing a pointer to some instance)

### Choosing which to use?

Already discussed class versus struct in previous lecture (also in your Reading Assignment).

Use of enum is situational (any time you have a type of data with discrete values).

## Methods

### Parameters Names

All parameters to all functions have an `internal` name and an `external` name 

The `internal` name is the name of the local variable you use inside the method

The `external` name is what callers use when tey call the method

You can put _ if you don't want callers to use an external name at all for a given parameter

This would almost never be done for anything but the first parameter.

If you only put one parameter name, it will be both the external and iternal name.

``` swift
func foo(externalFirst first: Int, externalSecond second: Double) {
    var sum = 0.0
    for _ in 0..<first {
        sum += secod
    }
}

for bar() {
    let result = foo(externalFirst: 123, externalSecond: 5.5)
}
```

### You can override methods/properties from your supper `class`

Precede your `func` or `var` with the keyword `override`

A method can be marked `final` which will prevent subclasses from being able to override

Entire classes can also be marked `final`

### Both types and instances can have methods/properties

Type methods and properties are denoted with the keyword `static`.

