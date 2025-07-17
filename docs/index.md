# Stria Language Specification

**Version:** 1.0  
**Date:** January 2025

## Table of Contents

1. [Introduction](#introduction)
2. [Language Overview](#language-overview)
3. [Lexical Structure](#lexical-structure)
4. [Types and Values](#types-and-values)
5. [Expressions and Operators](#expressions-and-operators)
6. [Statements and Control Flow](#statements-and-control-flow)
7. [Structs and Initialization](#structs-and-initialization)
8. [Functions and Methods](#functions-and-methods)
9. [Annotations](#annotations)
10. [Schema System](#schema-system)
11. [Standard Library](#standard-library)
12. [Error Handling](#error-handling)
13. [Future Features](#future-features)

---

## Introduction

Stria is a declarative, schema-validated configuration management language designed for creating robust and maintainable configuration systems. The language emphasizes type safety, immutability, and clear schema definitions to ensure configuration correctness.

### Etymology

The name "Stria" derives from three Latin and English roots:

- **Structura** (Latin): Structure
- **Via** (Latin): Path/Way
- **Strict** (English): Rigorous

Together, these represent the language's philosophy of providing a **strict structure** and **clear path** for configuration management.

### Design Goals

- **Type Safety**: All configurations are validated against strict schemas
- **Immutability**: Configurations are immutable by default
- **Expressiveness**: Rich syntax for complex configuration scenarios
- **Maintainability**: Clear, readable configuration files
- **Validation**: Comprehensive error checking and reporting

---

## Language Overview

### File Extension

Stria files use the `.stria` extension.

### Basic Structure

A Stria project consists of two main types of files:

1. **Schema Files**: Define the structure and validation rules
2. **Configuration Files**: Contain the actual configuration data

### Key Features

- **Struct-based Architecture**: Configuration is organized using `struct` definitions
- **Multiple Initialization Patterns**: Flexible object instantiation
- **Schema Validation**: Compile-time validation against schemas
- **Expression-oriented**: Everything is an expression that returns a value
- **Controlled Side Effects**: Limited to essential configuration operations

---

## Lexical Structure

### Comments

Stria supports three types of comments:

```stria
// Line comment
# Shell-style line comment
/*
 * Block comment
 */
```

### Identifiers

Identifiers follow standard conventions:

- Must start with a letter or underscore
- Can contain letters, digits, and underscores
- Case-sensitive

```stria
validIdentifier
_privateField
myVariable123
```

### Keywords

Reserved keywords in Stria:

```
struct    init      fun       val       var
if        else      true      false     null
this      get       repeated  private   use
schema    mixin     error     match     when
```

### Literals

#### String Literals

```stria
val simpleString = 'Hello, World!'
val doubleQuoted = "Hello, World!"
val templateString = `Hello, ${name}!`
```

#### Numeric Literals

```stria
val integer = 42
val decimal = 3.14
val negative = -123
```

#### Boolean Literals

```stria
val isTrue = true
val isFalse = false
```

#### List Literals

```stria
val numbers = [1, 2, 3, 4, 5]
val strings = ['a', 'b', 'c']
val mixed = [1, 'two', true]
```

---

## Types and Values

### Primitive Types

#### Basic Types

- `string`: Text data
- `int`: Integer numbers
- `float`: Floating-point numbers
- `bool`: Boolean values (`true` or `false`)

#### Collection Types

- `List<T>`: Ordered collection of elements
- `Range<T>`: Range of values (e.g., `1..10`)

#### Optional Types

Optional types are denoted with `?`:

```stria
val optionalString: string? = null
val optionalNumber: int? = 42
```

### Type Inference

Stria supports type inference:

```stria
val inferredString = 'Hello'  // string
val inferredNumber = 42       // int
val inferredBool = true       // bool
```

### Union Types

Union types allow multiple possible types:

```stria
union StringOrNumber = string | int
val value: StringOrNumber = 'hello'
```

---

## Expressions and Operators

### Arithmetic Operators

```stria
val addition = 5 + 3        // 8
val subtraction = 10 - 4    // 6
val multiplication = 6 * 7  // 42
val division = 15 / 3       // 5
val modulo = 17 % 5         // 2
val power = 2 ** 3          // 8
```

### Comparison Operators

```stria
val equal = 5 == 5          // true
val notEqual = 5 != 3       // true
val greater = 10 > 5        // true
val less = 3 < 7            // true
val greaterEqual = 5 >= 5   // true
val lessEqual = 4 <= 8      // true
```

### Logical Operators

```stria
val and = true && false     // false
val or = true || false      // true
val not = !true             // false
```

### Bitwise Operators

```stria
val bitwiseAnd = 5 & 3      // 1
val bitwiseOr = 5 | 3       // 7
val bitwiseXor = 5 ^ 3      // 6
val bitwiseNot = ~5         // -6
val leftShift = 5 << 1      // 10
val rightShift = 10 >> 1    // 5
```

### Assignment Operators

```stria
var x = 10
x += 5      // x = 15
x -= 3      // x = 12
x *= 2      // x = 24
x /= 4      // x = 6
x %= 5      // x = 1
```

### Member Access

```stria
val config = MyStruct { name = 'test' }
val name = config.name
```

### Null Assertion

```stria
val nullable: string? = getValue()
val notNull = nullable!  // Asserts non-null
```

### Range Operators

```stria
val range = 1..10           // Range from 1 to 10
val rangeWithStep = 1..10 step 2  // 1, 3, 5, 7, 9
```

### Spread Operator

```stria
val list1 = [1, 2, 3]
val list2 = [4, 5, 6]
val combined = [...list1, ...list2]  // [1, 2, 3, 4, 5, 6]
```

### Pipeline Operator

```stria
val result = value
    |> transform1
    |> transform2
    |> finalTransform
```

### Array Operations

```stria
val list = [1, 2, 3, 4, 5]
val element = list[2]       // 3
val slice = list[1..3]      // [2, 3, 4]
```

---

## Statements and Control Flow

### Variable Declarations

```stria
val immutableValue = 42        // Immutable
var mutableValue = 'hello'     // Mutable
```

### If Expressions

```stria
val result = if (condition) 'yes' else 'no'

val complex = if (x > 0) {
    'positive'
} else if (x < 0) {
    'negative'
} else {
    'zero'
}
```

### Lambda Expressions

```stria
val lambda = { x -> x * 2 }
val multiParam = { x, y -> x + y }
```

### Anonymous Functions

```stria
val anonymousFunction = fun(x: int): int {
    x * 2
}
```

---

## Structs and Initialization

### Basic Struct Definition

```stria
struct Person {
    name: string
    age: int
    email?: string  // Optional field
}
```

### Initialization Methods

#### Default Initialization

```stria
struct Point {
    init {
        x = 0
        y = 0
    }
    x: int
    y: int
}
```

#### Parameterized Initialization

```stria
struct Point {
    init(x: int, y: int) {
        this.x = x
        this.y = y
    }
    x: int
    y: int
}
```

#### Primary Constructor

```stria
struct Point {
    init(this.x, this.y)
    x: int
    y: int
}
```

### Multiple Initializers

```stria
struct Version {
    init {
        major = 0
        minor = 0
        patch = 1
    }

    init(major: int) {
        this.major = major
        minor = 0
        patch = 0
    }

    init(this.major, this.minor, this.patch)

    major: int
    minor: int
    patch: int
}
```

### Struct Instantiation

```stria
val point1 = Point()
val point2 = Point(10, 20)
val point3 = Point {
    x = 5
    y = 10
}
```

### Repeated Properties

```stria
struct Container {
    repeated items: string[] = []
}

val container = Container {
    items('first')
    items('second')
    items('third')
}
```

### Mixin Support

```stria
struct MixinExample {
    mixinField: string = 'default'
}

struct MainStruct {
    mixin MixinExample
    mainField: string
}
```

---

## Functions and Methods

### Function Declaration

```stria
fun add(a: int, b: int): int {
    a + b
}
```

### Method Declaration

```stria
struct Calculator {
    fun add(a: int, b: int): int {
        a + b
    }

    get currentValue(): int {
        // Getter method
        value
    }

    private fun validate() {
        // Private method
        if (value < 0) error('Invalid value')
    }

    value: int
}
```

### Function Call Rules

- Struct methods can be called only once by default
- Use `repeated` modifier for multiple calls
- Global/built-in functions can be called multiple times
- Getters can be called multiple times

```stria
struct Example {
    fun onceOnly() { /* ... */ }
    repeated fun multipleAllowed() { /* ... */ }
    get property(): string { /* ... */ }
}
```

### Lambda and Higher-Order Functions

```stria
val transform: (int) -> int = { x -> x * 2 }
val result = list.map(transform)
```

### Infix Functions

```stria
infix fun Int.add(other: Int): Int = this + other
val result = 5 add 3  // 8
```

---

## Annotations

### Built-in Annotations

#### @description

```stria
@description 'User configuration'
struct User {
    @description 'The user name'
    name: string
}
```

#### @name

```stria
struct Config {
    @name 'database_url'
    databaseUrl: string
}
```

#### @deprecated

```stria
struct LegacyConfig {
    @deprecated 'Use newField instead'
    oldField: string

    newField: string
}
```

#### @serialize

```stria
struct Version {
    @serialize 'toString'
    get toString(): string {
        `${major}.${minor}.${patch}`
    }

    major: int
    minor: int
    patch: int
}
```

#### @flatten

```stria
struct NestedConfig {
    @flatten
    database: DatabaseConfig
}
```

---

## Schema System

### Schema Declaration

```stria
schema {
    AppConfig
    DatabaseConfig
    ServerConfig
}
```

### Schema Reference

Configuration files reference schemas using the `#schema` directive:

```stria
#schema './schema.stria'

AppConfig {
    database = DatabaseConfig {
        host = 'localhost'
        port = 5432
    }
}
```

### Schema Validation

- All configuration files must reference a schema
- Schema files cannot contain configuration data
- Runtime validation ensures type safety
- Compilation errors for schema violations

---

## Standard Library

### Import System

```stria
use getEnv
use readFile
use writeFile
```

### Built-in Functions

#### Environment Functions

```stria
use getEnv
val path = getEnv('PATH')!
```

#### File Functions

```stria
use readFile
val content = readFile('config.txt')
```

#### Utility Functions

```stria
// String functions
val upper = str.toUpperCase()
val lower = str.toLowerCase()
val length = str.length()

// List functions
val size = list.size()
val first = list.first()
val last = list.last()
```

---

## Error Handling

### Error Function

```stria
fun validate(value: int) {
    if (value < 0) error('Value must be non-negative')
}
```

### Error Messages

Stria provides Rust-style error messages with clear descriptions and suggestions:

```
error[E001]: Type mismatch
  --> config.stria:5:10
   |
 5 |     port = "8080"
   |            ^^^^^^ expected int, found string
   |
help: convert string to int
   |
 5 |     port = 8080
   |            ^^^^
```

### Validation Patterns

```stria
struct Config {
    init {
        validate()
    }

    private fun validate() {
        if (port < 1 || port > 65535) {
            error('Port must be between 1 and 65535')
        }
    }

    port: int
}
```

---

## Future Features

### Planned Features

#### Type Constraints

```stria
type Port = int(min: 1, max: 65535)
type Email = string(format: 'email')
type Password = string(minLength: 8, maxLength: 64)
```

#### Match Expressions

```stria
val result = match value {
    is String -> "String value"
    is Int -> "Integer value"
    else -> "Other type"
}
```

#### Label and Break/Continue

```stria
outer@ for (i in 1..10) {
    for (j in 1..10) {
        if (condition) break@outer
    }
}
```

#### Regular Expressions

```stria
use regex
val pattern = regex(r"\d+")
val matches = pattern.findAll(text)
```

---

## Conclusion

Stria provides a powerful, type-safe configuration management system with a focus on clarity, validation, and maintainability. The language's design emphasizes immutability, expression-oriented programming, and comprehensive error handling to ensure robust configuration systems.

For implementation details and examples, refer to the test cases in the `testcases/` directory of the project repository.

---

_This specification is subject to change as the language evolves. Please refer to the latest version for the most current information._
