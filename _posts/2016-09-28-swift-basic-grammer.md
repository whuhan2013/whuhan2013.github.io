---
layout: post
title: swift基础语法
date: 2016-9-28
categories: blog
tags: [IOS]
description: IOS
---


**本文主要包括以下内容**

- Differentiate between a constant and a variable
- Know when to use implicit and when to use explicit type declarations
- Understand the advantage of using optionals and optional binding
- Differentiate between optionals and implicitly unwrapped optionals
- Understand the purpose of conditional statements and loops
- Use switch statements for conditional branching beyond a binary condition
- Use where clauses to impose additional constraints in conditional statements
- Differentiate between functions, methods, and initializers
- Differentiate between classes, structures, and enumerations
- Understand syntax for (and basic concepts behind) inheritance and protocol conformance
- Determine implicit types and find additional information using Xcode’s quick help shortcut (Option-click)
- Import and use UIKit
- designated init and convient init


**常量与变量**

```
var myVariable = 42
myVariable = 50
let myConstant = 42

let implicitInteger = 70
let implicitDouble = 70.0
let explicitDouble: Double = 70
```

**optionals使用**

Use optionals to work with values that might be missing. An optional value either contains a value or contains nil (no value) to indicate that a value is missing. Write a question mark (?) after the type of a value to mark the value as optional.

```
let optionalInt: Int? = 9
To get the underlying value from an optional, you unwrap it. You’ll learn unwrapping optionals later, but the most straightforward way to do it involves the force unwrap operator (!). Only use the unwrap operator if you’re sure the underlying value isn’t nil.

let actualInt: Int = optionalInt!
Optionals are pervasive in Swift, and are very useful for many situations where a value may or may not be present. They’re especially useful for attempted type conversions.

var myString = "7"
var possibleInt = Int(myString)
print(possibleInt)
```


**数组**

An array is a data type that keeps track of an ordered collection of items. Create arrays using brackets ([]), and access their elements by writing the index in brackets. Arrays start at index 0.

```
var ratingList = ["Poor", "Fine", "Good", "Excellent"]
ratingList[1] = "OK"
ratingList
To create an empty array, use the initializer syntax. You’ll learn more about initializers in a little while.

// Creates an empty array.
let emptyArray = [String]()
```


**Control Flow之if**

```
let number = 23
if number < 10 {
    print("The number is small")
} else if number > 100 {
    print("The number is pretty big")
} else {
    print("The number is between 10 and 100")
}
```

If the optional value is nil, the conditional is false, and the code in braces is skipped. Otherwise, the optional value is unwrapped and assigned to the constant after let, which makes the unwrapped value available inside the block of code.

You can use a single if statement to bind multiple values. A where clause can be added to a case to further scope the conditional statement. In this case, the if statement executes only if the binding is successful for all of these values and all conditions are met.

```
var optionalHello: String? = "Hello"
if let hello = optionalHello where hello.hasPrefix("H"), let name = optionalName {
    greeting = "\(hello), \(name)"
}
```


**switches**

Switches in Swift are quite powerful. A switch statement supports any kind of data and a wide variety of comparison operations—it isn’t limited to integers and tests for equality. In this example, the switch statement switches on the value of the vegetable string, comparing the value to each of its cases and executing the one that matches.


```
let vegetable = "red pepper"
switch vegetable {
case "celery":
    let vegetableComment = "Add some raisins and make ants on a log."
case "cucumber", "watercress":
    let vegetableComment = "That would make a good tea sandwich."
case let x where x.hasSuffix("pepper"):
    let vegetableComment = "Is it a spicy \(x)?"
default:
    let vegetableComment = "Everything tastes good in soup."
}
```

You can keep an index in a loop by using a Range. Use the half-open range operator ( ..<) to make a range of indexes.

```
var firstForLoop = 0
for i in 0..<4 {
    firstForLoop += i
}
print(firstForLoop)
```


**函数与方法**        

A function is a reusable, named piece of code that can be referred to from many places in a program.

Use func to declare a function. A function declaration can include zero or more parameters, written as name: Type, which are additional pieces of information that must be passed into the function when it’s called. Optionally, a function can have a return type, written after the ->, which indicates what the function returns as its result. A function’s implementation goes inside of a pair of curly braces ({}).

```
func greet(name: String, day: String) -> String {
    return "Hello \(name), today is \(day)."
}
```


**类与构造函数**

Use class followed by the class’s name to define a class. A property declaration in a class is written the same way as a constant or variable declaration, except that it’s in the context of a class. Likewise, method and function declarations are written the same way. This example declares a Shape class with a numberOfSides property and a simpleDescription() method.

```
class Shape {
    var numberOfSides = 0
    func simpleDescription() -> String {
        return "A shape with \(numberOfSides) sides."
    }
}
```

Create an instance of a class—an object—by putting parentheses after the class name. Use dot syntax to access the properties and methods of the instance. Here, shape is an object that’s an instance of the Shape class.

```
var shape = Shape()
shape.numberOfSides = 7
var shapeDescription = shape.simpleDescription()
```

This Shape class is missing something important: an initializer. An initializer is a method that prepares an instance of a class for use, which involves setting an initial value for each property and performing any other setup. Use init to create one. This example defines a new class, NamedShape, that has an initializer which takes in a name.

```
class NamedShape {
    var numberOfSides = 0
    var name: String
    
    init(name: String) {
        self.name = name
    }
    
    func simpleDescription() -> String {
        return "A shape with \(numberOfSides) sides."
    }
}
```

Notice how self is used to distinguish the name property from the name argument to the initializer. Every property needs a value assigned—either in its declaration (as with numberOfSides) or in the initializer (as with name).

You don’t call an initializer by writing init; you call it by putting parentheses with the appropriate arguments after the class name. When you call an initializer, you include all arguments names along with their values.

```
let namedShape = NamedShape(name: "my named shape")
```


**继承** 


```
class Square: NamedShape {
    var sideLength: Double
    
    init(sideLength: Double, name: String) {
        self.sideLength = sideLength
        super.init(name: name)
        numberOfSides = 4
    }
    
    func area() ->  Double {
        return sideLength * sideLength
    }
    
    override func simpleDescription() -> String {
        return "A square with sides of length \(sideLength)."
    }
}
let testSquare = Square(sideLength: 5.2, name: "my test square")
testSquare.area()
testSquare.simpleDescription()
```

Notice that the initializer for the Square class has three different steps:

－ Setting the value of properties that the subclass, Square, declares.
－ Calling the initializer of the superclass, NamedShape.
－ Changing the value of properties defined by the superclass, NamedShape. Any additional setup work that uses methods, getters, or setters can also be done at this point.



Sometimes, initialization of an object needs to fail, such as when the values supplied as the arguments are outside of a certain range, or when data that’s expected to be there is missing. Initializers that may fail to successfully initialize an object are called failable initializers. A failable initializer can return nil after initialization. Use init? to declare a failable initializer.

```
class Circle: NamedShape {
    var radius: Double
    
    init?(radius: Double, name: String) {
        self.radius = radius
        super.init(name: name)
        numberOfSides = 1
        if radius <= 0 {
            return nil
        }
    }
    
    override func simpleDescription() -> String {
        return "A circle with a radius of \(radius)."
    }
}
let successfulCircle = Circle(radius: 4.2, name: "successful circle")
let failedCircle = Circle(radius: -7, name: "failed circle")
```

**构造函数的一些关键词**   

The convenience keyword next to an initializer indicates a convenience initializer. Convenience initializers are secondary initializers. They can add additional behavior or customization, but must eventually call through to a designated initializer.

A required keyword next to an initializer indicates that every subclass of the class that has that initializer must implement its own version of the initializer (if it implements any initializer).



**Enumerations and Structures**


Classes aren’t the only ways to define data types in Swift. Enumerations and structures have similar capabilities to classes, but can be useful in different contexts.

Enumerations define a common type for a group of related values and enable you to work with those values in a type-safe way within your code. Enumerations can have methods associated with them.

Use enum to create an enumeration.


```
enum Rank: Int {
    case Ace = 1
    case Two, Three, Four, Five, Six, Seven, Eight, Nine, Ten
    case Jack, Queen, King
    func simpleDescription() -> String {
        switch self {
        case .Ace:
            return "ace"
        case .Jack:
            return "jack"
        case .Queen:
            return "queen"
        case .King:
            return "king"
        default:
            return String(self.rawValue)
        }
    }
}
let ace = Rank.Ace
let aceRawValue = ace.rawValue
```

In the example above, the raw-value type of the enumeration is Int, so you have to specify only the first raw value. The rest of the raw values are assigned in order. You can also use strings or floating-point numbers as the raw type of an enumeration. Use the rawValue property to access the raw value of an enumeration member.

Use the init?(rawValue:) initializer to make an instance of an enumeration from a raw value.

```
if let convertedRank = Rank(rawValue: 3) {
    let threeDescription = convertedRank.simpleDescription()
}
```

The member values of an enumeration are actual values, not just another way of writing their raw values. In fact, in cases where there isn’t a meaningful raw value, you don’t have to provide one.

```
enum Suit {
    case Spades, Hearts, Diamonds, Clubs
    func simpleDescription() -> String {
        switch self {
        case .Spades:
            return "spades"
        case .Hearts:
            return "hearts"
        case .Diamonds:
            return "diamonds"
        case .Clubs:
            return "clubs"
        }
    }
}
let hearts = Suit.Hearts
let heartsDescription = hearts.simpleDescription()
```


Notice the two ways that the Hearts member of the enumeration is referred to above: When a value is assigned to the hearts constant, the enumeration member Suit.Hearts is referred to by its full name because the constant doesn’t have an explicit type specified. Inside the switch, the enumeration member is referred to by the abbreviated form .Hearts because the value of self is already known to be a suit. You can use the abbreviated form anytime the value’s type is already known.


**structures**


Structures support many of the same behaviors as classes, including methods and initializers. One of the most important differences between structures and classes is that structures are always copied when they are passed around in your code, but classes are passed by reference. Structures are great for defining lightweight data types that don’t need to have capabilities like inheritance and type casting.

Use struct to create a structure.

```
struct Card {
    var rank: Rank
    var suit: Suit
    func simpleDescription() -> String {
        return "The \(rank.simpleDescription()) of \(suit.simpleDescription())"
    }
}
let threeOfSpades = Card(rank: .Three, suit: .Spades)
let threeOfSpadesDescription = threeOfSpades.simpleDescription()
```


不知道为什么前面加个点是什么意思


**Protocols**

A protocol defines a blueprint of methods, properties, and other requirements that suit a particular task or piece of functionality. The protocol doesn’t actually provide an implementation for any of these requirements—it only describes what an implementation will look like. The protocol can then be adopted by a class, structure, or enumeration to provide an actual implementation of those requirements. Any type that satisfies the requirements of a protocol is said to conform to that protocol.

Use protocol to declare a protocol.

```
protocol ExampleProtocol {
    var simpleDescription: String { get }
    func adjust()
}
```

Protocols can require that conforming types have specific instance properties, instance methods, type methods, operators, and subscripts. Protocols can require specific instance methods and type methods to be implemented by conforming types. These methods are written as part of the protocol’s definition in exactly the same way as for normal instance and type methods, but without curly braces or a method body.

Classes, structures, and enumerations adopt a protocol by listing its name after their name, separated by a colon. A type can adopt any number of protocols, which appear in a comma-separated list. If a class has a superclass, the superclass’s name must appear first in the list, followed by protocols. You conform to the protocol by implementing all of its requirements.

Here, SimpleClass adopts the ExampleProtocol protocol, and conforms to the protocol by implementing the simpleDescription property and adjust() method.

```
class SimpleClass: ExampleProtocol {
    var simpleDescription: String = "A very simple class."
    var anotherProperty: Int = 69105
    func adjust() {
        simpleDescription += "  Now 100% adjusted."
    }
}
var a = SimpleClass()
a.adjust()
let aDescription = a.simpleDescription
```

Protocols are first-class types, which means they can be treated like other named types. For example, you can create an ExampleProtocol array and call adjust() on each of the instances in it (because any instance in that array would be guaranteed to implement adjust(), one of the protocol’s requirements).

```
class SimpleClass2: ExampleProtocol {
    var simpleDescription: String = "Another very simple class."
    func adjust() {
        simpleDescription += "  Adjusted."
    }
}
 
var protocolArray: [ExampleProtocol] = [SimpleClass(), SimpleClass(), SimpleClass2()]
for instance in protocolArray {
    instance.adjust()
}
protocolArray
```


**Swift and Cocoa Touch**

Swift is designed to provide seamless interoperability with Cocoa Touch, the set of Apple frameworks you use to develop apps for iOS. As you walk through the rest of the lessons, it helps to have a basic understanding of how Swift interacts with Cocoa Touch.

So far, you’ve been working exclusively with data types from the Swift standard library. The Swift standard library is a set of data types and capabilities designed for Swift and baked into the language. Types like String and Array are examples of data types you see in the standard library.

```
let sampleString: String = "hello"
let sampleArray: Array = [1, 2, 3.1415, 23, 42]
```


When writing iOS apps, you’ll be using more than the Swift standard library. One of the most frequently used frameworks in iOS app development is UIKit. UIKit contains useful classes for working with the UI (user interface) layer of your app.

To get access to UIKit, simply import it as a module into any Swift file or playground

After importing UIKit, you can use Swift syntax with UIKit types and with their methods, properties, and so on.

```
let redSquare = UIView(frame: CGRect(x: 0, y: 0, width: 44, height: 44))
redSquare.backgroundColor = UIColor.redColor()
```

Many of the classes you’ll be introduced to in the lessons come from UIKit, so you’ll see this import statement often.


**参考:**[swift基础语法](https://developer.apple.com/library/content/referencelibrary/GettingStarted/DevelopiOSAppsSwift/)



**designated init and convient init**

By the time any init is done, all properties must have values (optionals can have the value nil) There are two types of inits in a class, convenience and designated (i.e. not convenience)
A designated init must (and can only) call a designated init that is in its immediate superclass You must initialize all properties introduced by your class before calling a superclass’s init
You must call a superclass’s init before you assign a value to an inherited property
A convenience init must (and can only) call a designated init in its own class
A convenience init may call a designated init indirectly (through another convenience init) A convenience init must call a designated init before it can set any property values
The calling of other inits must be complete before you can access properties or invoke methods

- Inheriting init       
If you do not implement any designated inits, you’ll inherit all of your superclass’s designateds If you override all of your superclass’s designated inits, you’ll inherit all its convenience inits If you implement no inits, you’ll inherit all of your superclass’s inits
Any init inherited by these rules qualifies to satisfy any of the rules on the previous slide

- Required init       
A class can mark one or more of its init methods as required
Any subclass must implement said init methods (though they can be inherited per above rules)

- Fiable init    
 If an init is declared with a ? (or !) after the word init, it returns an Optional





















