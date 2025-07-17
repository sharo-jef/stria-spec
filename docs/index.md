# Stria Language Specification

Version: none
Date: none

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
as        is
```

### Literals

#### String Literals

```stria
val simpleString = 'Hello, World!'
val doubleQuoted = "Hello, World!"
val templateString = `Hello, ${name}!`
```

#### Numeric Literals

Stria supports numeric literals with optional type suffixes:

```stria
// Type suffix notation
val integer = 42i32        // i32
val byte = 255u8           // u8
val long = 1000000i64      // i64
val float = 3.14f32        // f32
val double = 3.14159265359f64 // f64
val negative = -123i32     // i32

// Default types (without suffix)
val defaultInt = 42        // i32 (default for integers)
val defaultFloat = 3.14    // f64 (default for floating-point)
val defaultNegative = -123 // i32
```

##### Supported Type Suffixes

- **Integer types**: `i8`, `i16`, `i32`, `i64`, `u8`, `u16`, `u32`, `u64`
- **Floating-point types**: `f32`, `f64`
- **Default types**: `i32` for integers, `f64` for floating-point numbers

#### Boolean Literals

```stria
val isTrue = true
val isFalse = false
```

#### List Literals

```stria
val numbers = [1, 2, 3, 4, 5]            // List<i32>
val strings = ['a', 'b', 'c']            // List<string>
val mixed = [1, 'two', true]             // List<i32 | string | bool>
val empty: List<i32> = []                // Empty list with explicit type
```

---

## Types and Values

### Primitive Types

#### Basic Types

- `string`: Text data
- `i8`: 8-bit signed integer (-128 to 127)
- `i16`: 16-bit signed integer (-32,768 to 32,767)
- `i32`: 32-bit signed integer (-2,147,483,648 to 2,147,483,647)
- `i64`: 64-bit signed integer (-9,223,372,036,854,775,808 to 9,223,372,036,854,775,807)
- `u8`: 8-bit unsigned integer (0 to 255)
- `u16`: 16-bit unsigned integer (0 to 65,535)
- `u32`: 32-bit unsigned integer (0 to 4,294,967,295)
- `u64`: 64-bit unsigned integer (0 to 18,446,744,073,709,551,615)
- `f32`: 32-bit IEEE 754 floating-point number
- `f64`: 64-bit IEEE 754 floating-point number
- `bool`: Boolean values (`true` or `false`)

#### Collection Types

- `List<T>`: Immutable ordered collection of elements
- `I32Range`: Range of i32 values with exclusive upper bound (e.g., `1..10`)
- `F64Range`: Range of f64 values with exclusive upper bound (e.g., `1.0..10.0`)

##### List<T> Type

The `List<T>` type represents an ordered collection of elements of type `T`. For configuration management convenience, lists are **mutable during construction** and become **immutable after execution completes**. List literals are created using square bracket notation:

```stria
val numbers: List<i32> = [1, 2, 3, 4, 5]
val strings: List<string> = ['hello', 'world']
val mixed: List<string | i32> = ['text', 42, 'more text']
val empty: List<i32> = []
```

###### Mutability Model

Lists in Stria follow a **build-time mutable, runtime immutable** model:

- **During configuration construction**: Lists can be modified using methods like `push()`, `pop()`, `insert()`, `remove()`
- **After execution completes**: Lists become immutable and cannot be modified
- **Type safety**: All operations are type-safe and checked at compile time

```stria
val servers = List<string>()
servers.push('web1.example.com')    // Valid during construction
servers.push('web2.example.com')    // Valid during construction
servers[0] = 'api.example.com'      // Valid during construction
// After execution: servers becomes immutable
```

###### Type Inference

List literals support type inference:

```stria
val inferredNumbers = [1, 2, 3]          // List<i32>
val inferredStrings = ['a', 'b', 'c']    // List<string>
val inferredMixed = [1, 'two', true]     // List<i32 | string | bool>
```

###### Array Syntax Shorthand

In certain contexts, `Type[]` serves as shorthand for `List<Type>`:

```stria
// These are equivalent:
val items1: List<string> = ['a', 'b', 'c']
val items2: string[] = ['a', 'b', 'c']

// In struct definitions:
struct Container {
    repeated items: string[] = []  // Equivalent to List<string>
}
```

###### Configuration Building Examples

The mutable-during-construction model makes configuration building intuitive:

```stria
// Building server configurations
val servers = List<ServerConfig>()
servers.push(ServerConfig { host = 'web1.example.com', port = 80 })
servers.push(ServerConfig { host = 'web2.example.com', port = 80 })

// Conditional configuration
if (enableHttps) {
    servers.forEach { it.port = 443 }
    servers.push(ServerConfig { host = 'ssl.example.com', port = 443 })
}

// Environment-specific adjustments
if (isDevelopment) {
    servers.clear()
    servers.push(ServerConfig { host = 'localhost', port = 8080 })
}

// Functional transformations
val httpsServers = servers.filter { it.port == 443 }
val hostnames = servers.map { it.host }
val isAllSecure = servers.all { it.port == 443 }

// Database configuration with dynamic setup
struct DatabaseConfig {
    repeated hosts: string[] = []

    init {
        // Add primary host
        hosts.push('primary.db.example.com')

        // Add replicas based on environment
        if (isProduction) {
            hosts.extend(['replica1.db.example.com', 'replica2.db.example.com'])
        } else {
            hosts.push('dev.db.example.com')
        }

        // Remove duplicates and sort for consistency
        hosts = hosts.distinct().sorted()
    }
}

// Configuration with filtering and deduplication
struct ServiceConfig {
    repeated endpoints: string[] = []

    init {
        // Add various endpoints
        endpoints.push('api.example.com')
        endpoints.push('api.example.com')  // Duplicate
        endpoints.push('beta.api.example.com')
        endpoints.push('admin.api.example.com')

        // Remove duplicates
        endpoints = endpoints.distinct()

        // Filter out beta endpoints in production
        if (isProduction) {
            endpoints = endpoints.filter { !it.contains('beta') }
        }

        // Sort for consistency
        endpoints = endpoints.sorted()
    }
}
```

#### Optional Types

Optional types are denoted with `?`:

```stria
val optionalString: string? = null
val optionalNumber: i32? = 42
```

### Type Inference

Stria supports type inference with the following default types:

```stria
val inferredString = 'Hello'  // string
val inferredInt = 42          // i32 (default for integers)
val inferredFloat = 3.14      // f64 (default for floating-point)
val inferredBool = true       // bool

// Explicit type annotation
val explicitInt: i64 = 42     // i64
val explicitFloat: f32 = 3.14 // f32
```

#### Default Type Rules

- **Integer literals**: Default to `i32` when no type suffix is provided
- **Floating-point literals**: Default to `f64` when no type suffix is provided
- **Type suffixes**: Override default types (e.g., `42i64`, `3.14f32`)
- **Type annotations**: Override both defaults and suffixes

### Union Types

Union types allow multiple possible types:

```stria
union StringOrNumber = string | i32
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
val range = 1..10           // Range from 1 to 10 (exclusive upper bound)
val inclusiveRange = 1..=10 // Range from 1 to 10 (inclusive)
val rangeWithStep = 1..10 step 2  // 1, 3, 5, 7, 9
```

#### Range Types

Range expressions are interpreted as `RangeExpression` at compile time and handled as `I32Range` or `F64Range` at runtime:

```stria
struct I32Range {
    from: i32
    to: i32
    step: i32 = 1
}

struct F64Range {
    from: f64
    to: f64
    step: f64 = 1.0
}
```

#### Range Methods

Range types support the following infix functions:

```stria
// Step function for I32Range
infix fun I32Range.step(step: i32): I32Range {
    val {from, to} = this
    I32Range {
        this.from = from
        this.to = to
        this.step = if (step == 0) {
            error('step must not be zero')
        } else if ((step < 0 && from < to) || (step > 0 && from > to)) {
            -step
        } else {
            step
        }
    }
}

// Contains function for I32Range
infix fun i32.in(range: I32Range): bool {
    if (range.step > 0) {
        this >= range.from && this <= range.to && ((this - range.from) % range.step == 0)
    } else {
        this <= range.from && this >= range.to && ((range.from - this) % (-range.step) == 0)
    }
}

// Until function for integers (exclusive upper bound)
infix fun i32.until(to: i32): I32Range {
    if (to == this) error('to must not be equal to from')
    if (to < this) error('to must be greater than from')
    I32Range {
        from = this
        this.to = to - 1
        step = 1
    }
}

// Down to function for integers (descending range)
infix fun i32.downTo(to: i32): I32Range {
    if (to == this) error('to must not be equal to from')
    if (to > this) error('to must be less than from')
    I32Range {
        from = this
        this.to = to
        step = -1
    }
}

// Step function for F64Range
infix fun F64Range.step(step: f64): F64Range {
    val {from, to} = this
    F64Range {
        this.from = from
        this.to = to
        this.step = if (step == 0) {
            error('step must not be zero')
        } else if ((step < 0 && from < to) || (step > 0 && from > to)) {
            -step
        } else {
            step
        }
    }
}

// Contains function for F64Range
infix fun f64.in(range: F64Range): bool {
    if (range.step > 0) {
        this >= range.from && this <= range.to && ((this - range.from) % range.step == 0)
    } else {
        this <= range.from && this >= range.to && ((range.from - this) % (-range.step) == 0)
    }
}

// Until function for floats (exclusive upper bound)
infix fun f64.until(to: f64): F64Range {
    if (to == this) error('to must not be equal to from')
    if (to < this) error('to must be greater than from')
    F64Range {
        from = this
        this.to = to
        step = 1.0
    }
}

// Down to function for floats (descending range)
infix fun f64.downTo(to: f64): F64Range {
    if (to == this) error('to must not be equal to from')
    if (to > this) error('to must be less than from')
    F64Range {
        from = this
        this.to = to
        step = -1.0
    }
}
```

#### Range Usage Examples

```stria
// Basic ranges
val numbers = 1..10          // I32Range from 1 to 10 (exclusive upper bound)
val floats = 1.0..10.0       // F64Range from 1.0 to 10.0 (exclusive upper bound)
val inclusive = 1..=10       // I32Range from 1 to 10 (inclusive)

// Range with step
val evens = 2..20 step 2     // 2, 4, 6, 8, 10, 12, 14, 16, 18, 20
val halves = 0.5..5.0 step 0.5  // 0.5, 1.0, 1.5, 2.0, 2.5, 3.0, 3.5, 4.0, 4.5, 5.0

// Until ranges (exclusive upper bound)
val lessThan10 = 1 until 10  // 1, 2, 3, 4, 5, 6, 7, 8, 9

// Descending ranges
val countdown = 10 downTo 1  // 10, 9, 8, 7, 6, 5, 4, 3, 2, 1

// Range membership testing
val inRange = 10 in 1..10     // false (10 is not included in 1..10)
val inInclusiveRange = 10 in 1..=10 // true (10 is included in 1..=10)
val notInRange = 15 in 1..10 // false
val inSteppedRange = 6 in 2..20 step 2  // true (6 is even)
val notInSteppedRange = 7 in 2..20 step 2  // false (7 is not even)

// Float range membership testing with step
val inFloatRange = 4.0 in 2.0..20.0 step 2.0  // true (4.0 is in the sequence)
val notInFloatRange = 3.0 in 2.0..20.0 step 2.0  // false (3.0 is not in the sequence)
val inFloatSteppedRange = 2.5 in 1.0..5.0 step 0.5  // true (2.5 is in the sequence)
val notInFloatSteppedRange = 2.3 in 1.0..5.0 step 0.5  // false (2.3 is not in the sequence)
```

### Spread Operator

```stria
val list1 = [1, 2, 3]
val list2 = [4, 5, 6]
val combined = [...list1, ...list2]  // [1, 2, 3, 4, 5, 6]
```

### Type Cast Operators

Stria supports explicit type casting using the `as` keyword:

```stria
val intValue = 42i32
val floatValue = intValue as f64    // 42.0
val stringValue = intValue as string // "42"
val boolValue = intValue as bool     // true (non-zero values are true)

val floatToInt = 3.14f64 as i32     // 3 (truncated)
val stringToInt = "123" as i32      // 123
val boolToInt = true as i32         // 1
```

#### Supported Cast Operations

- **Numeric casts**: Between integer and floating-point types
- **String casts**: From any type to string representation
- **Boolean casts**: From numeric types (0 is false, non-zero is true)
- **Parsing casts**: From string to numeric or boolean types

#### Cast Safety

Type casts may truncate or lose precision:

```stria
val largeInt = 1000i32
val smallInt = largeInt as u8       // 232 (wraps around)
val floatPrecision = 3.14159f64 as f32  // May lose precision
```

### List Operations

```stria
val list = [1, 2, 3, 4, 5]

// Index access
val element = list[2]       // 3
val slice = list[1..3]      // [2, 3] (exclusive upper bound)
val inclusiveSlice = list[1..=3]  // [2, 3, 4] (inclusive)

// Mutation during construction
list.push(6)                // [1, 2, 3, 4, 5, 6]
list[0] = 10                // [10, 2, 3, 4, 5, 6]
list.insert(1, 20)          // [10, 20, 2, 3, 4, 5, 6]

// Properties and methods
val size = list.size()      // 7
val isEmpty = list.isEmpty() // false
val contains = list.contains(3) // true

// Functional operations
val doubled = list.map { it * 2 }     // [20, 40, 4, 6, 8, 10, 12]
val evens = list.filter { it % 2 == 0 } // [10, 20, 2, 4, 6]
val sum = list.reduce { acc, it -> acc + it } // 45

// Distinct and sorting (useful for configuration)
val duplicates = [1, 2, 2, 3, 1, 4]
val unique = duplicates.distinct()    // [1, 2, 3, 4]
val sorted = unique.sorted()          // [1, 2, 3, 4]

// List concatenation
val other = [7, 8, 9]
val combined = list + other  // [10, 20, 2, 3, 4, 5, 6, 7, 8, 9]
val spread = [...list, 100, ...other] // [10, 20, 2, 3, 4, 5, 6, 100, 7, 8, 9]
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
// Basic lambda expressions
val double = { x -> x * 2 }
val add = { x, y -> x + y }
val greeting = { -> 'Hello, World!' }

// Lambda expressions with explicit return
val doubleWithReturn = { x -> return x * 2 }
val addWithReturn = { x, y -> return x + y }
val greetingWithReturn = { -> return 'Hello, World!' }

// Lambda expressions with no return value
val doNothing = { -> }
val emptyLambda = {}

// Type-annotated lambda expressions
val typedDouble: (i32) -> i32 = { x -> x * 2 }
val typedAdd: (i32, i32) -> i32 = { x, y -> x + y }
val typedGreeting: () -> string = { -> 'Hello, World!' }

// Type-annotated lambda expressions with explicit return
val typedDoubleWithReturn: (i32) -> i32 = { x -> return x * 2 }
val typedAddWithReturn: (i32, i32) -> i32 = { x, y -> return x + y }
val typedGreetingWithReturn: () -> string = { -> return 'Hello, World!' }

// Type-annotated lambda expressions with void return
val typedDoNothing: () -> void = { -> }
val typedEmptyLambda: () -> void = {}
```

### Match Expressions

Match expressions provide pattern matching capabilities with type checking:

```stria
val result = match value {
    is string -> "String value"
    is i32 -> "Integer value"
    is bool -> "Boolean value"
    else -> "Other type"
}
```

#### Pattern Matching with Values

```stria
val status = match response.code {
    200 -> "OK"
    404 -> "Not Found"
    500 -> "Internal Server Error"
    else -> "Unknown Status"
}
```

#### Pattern Matching with Type Guards

```stria
union Value = string | i32 | bool

val description = match value {
    is string -> `String: ${value}`
    is i32 -> `Integer: ${value}`
    is bool -> `Boolean: ${value}`
}
```

The `is` operator performs runtime type checking and type narrowing:

- **Type Checking**: Returns `true` if the value is of the specified type
- **Type Narrowing**: Within the matching branch, the value is automatically cast to the checked type
- **Union Type Support**: Works with union types to discriminate between different type variants
- **Exhaustiveness**: When used with union types, the compiler ensures all possible types are handled

```stria
// Type checking examples
val isString = value is string     // Runtime type check
val isNumber = value is i32        // Returns true for i32 values
val isOptional = value is string?  // Checks for optional string type

// Type narrowing in conditionals
if (value is string) {
    // Within this block, 'value' is automatically treated as string
    val length = value.length()
}
```

#### Pattern Matching with Ranges

```stria
val category = match age {
    0..12 -> "Child"
    13..19 -> "Teen"
    20..64 -> "Adult"
    65..150 -> "Senior"
    else -> "Invalid age"
}

val grade = match score {
    90..100 -> "A"
    80..89 -> "B"
    70..79 -> "C"
    60..69 -> "D"
    0..59 -> "F"
    else -> "Invalid score"
}
```

Range patterns in match expressions are evaluated using the `in` operator internally. For example, `0..12` is equivalent to `score in 0..12`.

#### Match Expression Rules

- Match expressions must be exhaustive (cover all possible cases)
- The `else` clause can be used as a catch-all
- Each branch must return the same type
- Patterns are evaluated in order from top to bottom
- The first matching pattern is executed
- Range patterns are evaluated using the `in` operator internally

### Anonymous Functions

```stria
val anonymousFunction = fun(x: i32): i32 {
    x * 2
}
```

---

## Structs and Initialization

### Basic Struct Definition

```stria
struct Person {
    name: string
    age: u8
    email?: string  // Optional field
}
```

### Property Assignment Requirements

As a configuration description language, Stria has unique property assignment requirements that differ from typical programming languages:

#### Required Properties

- Properties without the `?:` modifier are considered required
- Required properties must be assigned at least once before execution completes
- Properties do not need to be assigned during initialization if they are assigned later during execution
- At execution completion, all required properties must have been assigned at least once

#### Optional Properties

- Properties with the `?:` modifier are considered optional
- Optional properties are implicitly initialized with `null` if not explicitly assigned
- Optional properties can remain `null` throughout execution without causing errors
- Optional properties can be assigned a value at any time during execution

#### Assignment Validation

```stria
struct DatabaseConfig {
    host: string      // Required - must be assigned before execution completes
    port: u16         // Required - must be assigned before execution completes
    username: string  // Required - must be assigned before execution completes
    password?: string // Optional - can remain unassigned
}

// Valid: All required properties assigned during initialization
val config1 = DatabaseConfig {
    host = 'localhost'
    port = 5432
    username = 'admin'
}

// Also valid: Required properties assigned later during execution
val config2 = DatabaseConfig {}
config2.host = 'localhost'
config2.port = 5432
config2.username = 'admin'
// password remains null (implicitly initialized)
// Execution completes successfully as all required properties are assigned

// Valid: Optional property explicitly assigned
val config3 = DatabaseConfig {
    host = 'localhost'
    port = 5432
    username = 'admin'
    password = 'secret123'  // Optional property explicitly assigned
}

// Valid: Optional property accessed as null
val config4 = DatabaseConfig {
    host = 'localhost'
    port = 5432
    username = 'admin'
}
val hasPassword = config4.password != null  // false, password is null

// Invalid: Would cause runtime error
val config5 = DatabaseConfig {
    host = 'localhost'
    port = 5432
    // username not assigned - runtime error at execution completion
}
```

#### Execution Completion Validation

- The Stria runtime validates that all required properties have been assigned before execution completes
- Unassigned required properties result in a runtime error with clear indication of which properties are missing
- This validation occurs after all configuration processing is complete

### Initialization Methods

#### Default Initialization

```stria
struct Point {
    init {
        x = 0
        y = 0
    }
    x: i32
    y: i32
}
```

For default initialization, the execution order is:

1. **Postfix Lambda**: Execution of the implicit postfix lambda block (if provided during instantiation)
2. **Initialization Block**: Execution of the `init` block defined in the struct

#### Parameterized Initialization

```stria
struct Point {
    init(x: i32, y: i32) {
        this.x = x
        this.y = y
    }
    x: i32
    y: i32
}
```

For parameterized initialization, the execution order is:

1. **Postfix Lambda**: Execution of the implicit postfix lambda block (if provided during instantiation)
2. **Initialization Block**: Execution of the `init` block defined in the struct

#### Primary Constructor

```stria
struct Point {
    init(this.x, this.y)
    x: i32
    y: i32
}
```

#### Primary Constructor with Initialization Block

Primary constructors can also include an initialization block for additional setup:

```stria
struct Point {
    init(this.x, this.y) {
        originDistance = math.sqrt(x ** 2 + y ** 2)
    }
    x: i32
    y: i32
    originDistance: f64
}

// Configuration example
struct ServerConfig {
    init(this.host, this.port) {
        url = `http://${host}:${port}`
        isSecure = port == 443
    }
    host: string
    port: u16
    url: string
    isSecure: bool
}
```

#### Primary Constructor Execution Order

Primary constructors execute in the following order:

1. **Parameter Assignment**: Direct assignment of constructor parameters (e.g., `this.x`, `this.y`)
2. **Postfix Lambda**: Execution of the implicit postfix lambda block (if provided during instantiation)
3. **Initialization Block**: Execution of the `init` block defined in the struct

```stria
struct Example {
    init(this.value) {
        // Step 3: This runs after parameter assignment and postfix lambda
        processed = value * 2
    }
    value: i32
    processed: i32
}

// Instantiation example showing execution order:
val example = Example(42) {
    // Step 2: This postfix lambda runs after parameter assignment
    value = value + 1  // value becomes 43
}
// Step 1: value = 42 (parameter assignment)
// Step 2: value = 43 (postfix lambda execution)
// Step 3: processed = 86 (init block execution: 43 * 2)
```

### Multiple Initializers

```stria
struct Version {
    init {
        major = 0
        minor = 0
        patch = 1
    }

    init(major: u16) {
        this.major = major
        minor = 0
        patch = 0
    }

    init(this.major, this.minor, this.patch)

    major: u16
    minor: u16
    patch: u16
}
```

### Struct Instantiation

Struct instantiation in Stria supports multiple syntaxes with sophisticated initialization mechanisms based on implicit lambda arguments.

#### Basic Instantiation

```stria
val point1 = Point()                    // Default constructor
val point2 = Point(10, 20)              // Parameterized constructor
val point3 = Point {                    // Block initialization
    x = 5
    y = 10
}
```

#### Block Initialization Syntax

The block initialization syntax `Point { ... }` is actually a shorthand for `Point() { ... }`:

```stria
// These are equivalent:
val point1 = Point { x = 5; y = 10 }
val point2 = Point() { x = 5; y = 10 }
```

#### Postfix Lambda Semantics

Similar to Kotlin's postfix lambdas, the block initialization can also be written as:

```stria
val point = Point({ -> x = 5; y = 10 })
```

This reveals that struct constructors implicitly have a final parameter of type `Point.() -> Unit` (using Kotlin-style notation).

#### Implicit Lambda Arguments in Constructors

All struct constructors, even the most abbreviated forms, implicitly accept a lambda parameter as their final argument:

```stria
// This apparent no-argument constructor:
struct Point {
    init { x = 0; y = 0 }
    x: i32
    y: i32
}

// Is actually equivalent to (shown as Kotlin-style pseudocode):
struct Point {
    init(action: Point.() -> Unit) {  // Kotlin-style pseudocode - not valid Stria syntax
        Point.action()  // Execute the lambda in the context of this instance
        x = 0
        y = 0
    }
    x: i32
    y: i32
}
```

#### Syntax Requirements

To distinguish struct instantiation from variable references, instantiation requires explicit constructor syntax:

```stria
// Required syntax:
val point1 = Point()    // Explicit constructor call
val point2 = Point{}    // Block initialization (shorthand for Point(){})

// Invalid syntax:
val point3 = Point      // This would be a variable reference, not instantiation
```

#### Optional Lambda Arguments

Constructor calls have an exceptional rule: the final lambda argument can be omitted entirely:

```stria
// These are all valid:
val point1 = Point(1, 2)        // No lambda provided
val point2 = Point(1, 2){}      // Empty lambda provided
val point3 = Point(1, 2){       // Lambda with initialization
    // Additional setup
}
```

This exception prevents the need to always provide a trailing lambda like `Point(1, 2){}` when no additional initialization is needed. This rule applies **only** to constructor calls and is not applicable to regular function calls.

#### Complex Initialization Examples

```stria
// Combining parameters and block initialization
val server = ServerConfig("localhost", 8080) {
    timeout = 30
    maxConnections = 100
}

// Multiple initialization steps
val database = DatabaseConfig {
    host = "db.example.com"
    port = 5432
    credentials {
        username = "admin"
        password = getEnv("DB_PASSWORD")!
    }
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

// Access the underlying list
val itemsList = container.items  // List<string>
itemsList.push('fourth')         // Add during construction
itemsList.remove('second')       // Remove during construction
val count = itemsList.size()     // 3
```

#### Repeated Properties with Constructor Selection

The `repeated` keyword supports constructor selection based on property names, allowing different constructors to be called for each named invocation:

```stria
struct Pipeline {
    repeated steps: Step[] {
        build -> BuildStep
        test -> TestStep
    }
}

union Step = BuildStep | TestStep

struct BuildStep {
    command: string
}

struct TestStep {
    script: string
}

val pipeline = Pipeline {
    build {
        command = "go build"
    }
    test {
        script = "go test"
    }
}
```

#### Repeated Properties Syntax

- **Basic repeated property**: `repeated propertyName: Type[] = []`
- **Constructor selection**: `repeated propertyName: Type[] { name1 -> Constructor1, name2 -> Constructor2 }`
- **Property access**: Each named constructor call adds an instance to the array

#### Multiple Calls to Same Constructor

```stria
struct TestSuite {
    repeated tests: Test[] {
        unit -> UnitTest
        integration -> IntegrationTest
    }
}

struct UnitTest {
    name: string
}

struct IntegrationTest {
    name: string
    timeout: u32
}

val suite = TestSuite {
    unit {
        name = "test_user_creation"
    }
    unit {
        name = "test_user_validation"
    }
    integration {
        name = "test_api_flow"
        timeout = 30
    }
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
fun add(a: i32, b: i32): i32 {
    a + b
}

// Function with no return value (void return type is optional)
fun printMessage(message: string) {
    // Implementation here
}

// Explicit void return type (optional)
fun printMessageExplicit(message: string): void {
    // Implementation here
}
```

#### Return Types

Functions can have explicit return types or omit them:

- **Explicit return type**: `fun functionName(): ReturnType { ... }`
- **Omitted return type**: `fun functionName() { ... }` (implicitly returns `void`)
- **Void return type**: The `: void` annotation is optional for functions that don't return a value

### Method Declaration

```stria
struct Calculator {
    fun add(a: i32, b: i32): i32 {
        a + b
    }

    get currentValue(): i32 {
        // Getter method
        value
    }

    private fun validate() {
        // Private method
        if (value < 0) error('Invalid value')
    }

    value: i32
}
```

### Method Call Rules

- Struct methods can be called only once by default
- Global/built-in functions can be called multiple times
- Getters can be called multiple times

```stria
struct Example {
    fun onceOnly() { /* ... */ }
    get property(): string { /* ... */ }
}
```

### Lambda and Higher-Order Functions

```stria
val transform: (i32) -> i32 = { x -> x * 2 }
val result = list.map(transform)
```

### Infix Functions

```stria
infix fun i32.add(other: i32): i32 {
    this + other
}
val result = 5 add 3  // 8
```

---

## Annotations

### Annotation Syntax

Annotations can be applied to structs, properties, and getters. The syntax supports optional parentheses for arguments:

```stria
@annotation              // No arguments
@annotation()            // No arguments (explicit)
@annotation 'value'      // Single argument
@annotation('value')     // Single argument (explicit)
@annotation('a', 'b')    // Multiple arguments (parentheses required)
```

### Built-in Annotations

#### @description

Provides documentation strings for IDE hover tooltips and automatic documentation generation.

```stria
@description 'User configuration structure'
struct User {
    @description 'The user name (required)'
    name: string

    @description 'The user age in years'
    age: u8

    @description 'Optional email address for notifications'
    email?: string
}
```

**Usage:**

- IDE hover tooltips display the description text
- Documentation generators use this for API documentation
- Can be applied to structs, properties, and getters

#### @name

Specifies the name to use when serializing to JSON or other formats. This allows Stria property names to differ from their serialized representation.

```stria
struct Config {
    @name 'database_url'
    databaseUrl: string

    @name 'api_key'
    apiKey: string

    @name 'max_connections'
    maxConnections: u32
}
```

**Usage:**

- Controls JSON property names during serialization
- Enables camelCase Stria properties with snake_case JSON output
- Useful for maintaining compatibility with existing APIs

#### @deprecated

Marks structs or properties as deprecated, providing migration guidance for IDE warnings.

```stria
struct LegacyConfig {
    @deprecated 'Use newField instead'
    oldField: string

    newField: string
}

@deprecated 'Use ModernConfig instead'
struct OldConfig {
    value: string
}
```

**Usage:**

- IDE shows deprecation warnings with the provided message
- Helps guide users to updated APIs
- Can be applied to structs, properties, and getters

#### @serialize

Controls how objects are serialized by specifying which getter to use for the serialized value.

##### On Getters

When applied to a getter, the getter's return value at program completion becomes the serialized value:

```stria
struct Version {
    major: u16
    minor: u16
    patch: u16

    @serialize
    get toString(): string {
        `${major}.${minor}.${patch}`
    }
}
```

**Example:**

```stria
// schema.stria
schema {
    AppConfig
}

struct AppConfig {
    version: Version
}

struct Version {
    major: u16
    minor: u16
    patch: u16

    @serialize
    get toString(): string {
        `${major}.${minor}.${patch}`
    }
}

// config.stria
#schema "./schema.stria"
AppConfig {
    version = Version {
        major = 1
        minor = 2
        patch = 3
    }
}
```

**Serialized output:**

```json
{
  "AppConfig": {
    "version": "1.2.3"
  }
}
```

##### On Structs

When applied to a struct, specify which getter to use for serialization:

```stria
@serialize 'toString'
struct Version {
    major: u16
    minor: u16
    patch: u16

    get toString(): string {
        `${major}.${minor}.${patch}`
    }
}
```

**Usage:**

- Transforms complex objects into simple serialized values
- Getter is evaluated at program completion
- Useful for computed values and string representations

#### @flatten

Flattens nested structures during JSON serialization, bringing child properties up to the parent level.

```stria
struct DatabaseConfig {
    host: string
    port: u16
    username: string
}

struct AppConfig {
    @flatten
    database: DatabaseConfig

    appName: string
}
```

**Example:**

```stria
// Without @flatten, would serialize as:
{
    "database": {
        "host": "localhost",
        "port": 5432,
        "username": "admin"
    },
    "appName": "MyApp"
}

// With @flatten, serializes as:
{
    "host": "localhost",
    "port": 5432,
    "username": "admin",
    "appName": "MyApp"
}
```

**Usage:**

- Reduces nesting in serialized output
- Useful for configuration flattening
- Can be applied to struct properties

---

## Schema System

### Schema Declaration

Schema files define the structure and validation rules using the `schema` declaration. These files cannot contain configuration data or use the `#schema` directive:

```stria
schema {
    AppConfig
    DatabaseConfig
    ServerConfig
}
```

### Schema Reference

Configuration files reference schemas using the `#schema` directive. These files cannot contain schema declarations:

```stria
#schema './schema.stria'

AppConfig {
    database = DatabaseConfig {
        host = 'localhost'
        port = 5432
    }
}
```

### File Type Restrictions

Both schema files and configuration files use the `.stria` extension, but they serve different purposes with mutual exclusivity:

- **Schema Files**:

  - Must contain a `schema` declaration
  - Cannot use the `#schema` directive
  - Cannot contain configuration data
  - Define struct types and validation rules

- **Configuration Files**:
  - Must use the `#schema` directive to reference a schema
  - Cannot contain `schema` declarations
  - Contain the actual configuration data
  - Must instantiate structs defined in the referenced schema

### Schema Validation

- All configuration files must reference a schema
- Schema files cannot contain configuration data
- Runtime validation ensures type safety
- Compilation errors for schema violations

---

## Standard Library

### Import System

The `use` statement is used to import standard library functions that are not built-in but are provided as standard functionality. These functions require explicit import before use.

```stria
use getEnv
use random
use print
```

#### Import Syntax

Functions are imported by specifying their exact name:

```stria
use functionName
```

Multiple functions can be imported with separate `use` statements:

```stria
use getEnv
use random
use print
```

### Standard Library Functions

The following functions are available through the `use` statement:

#### Environment Functions

```stria
use getEnv

fun getEnv(key: string): string? {
    // Returns environment variable value or null if not found
}

// Usage example:
val path = getEnv('PATH')!
val home = getEnv('HOME') ?: '/default/home'
```

#### Utility Functions

```stria
use random

fun random(): f64 {
    // Returns random f64 value between 0.0 and 1.0
}

// Usage example:
val randomValue = random()
val randomInt = (random() * 100) as i32
```

#### Output Functions

```stria
use print

fun print(message: any) {
    // Prints message to standard output
    // Note: 'any' type does not exist in Stria's type system
    // This function exceptionally accepts any type through special compiler handling
}

// Usage example:
print('Hello, World!')
print(42)
print(myStruct)
```

### Built-in Functions

The following functions are implicitly defined and available without import:

#### Error Functions

```stria
fun error(message: string): void {
    // Terminates execution with the given error message
}
```

#### Math Functions

Math-related functions and constants are available globally:

##### Math Constants

```stria
// Math constants
val pi: f64 = 3.141592653589793
val pi_f32: f32 = 3.141592653589793f32
```

##### Functions Available for All Numeric Types

The following functions are available as static methods on all numeric types (`i8`, `i16`, `i32`, `i64`, `u8`, `u16`, `u32`, `u64`, `f32`, `f64`):

```stria
// Comparison functions (available for all numeric types)
fun i32.max(a: i32, b: i32): i32 {
    if (a > b) a else b
}

fun i32.min(a: i32, b: i32): i32 {
    if (a < b) a else b
}

fun i32.abs(value: i32): i32 {
    if (value < 0) -value else value
}

// Similar functions exist for all other numeric types:
// i8.max, i8.min, i8.abs
// i16.max, i16.min, i16.abs
// i64.max, i64.min, i64.abs
// u8.max, u8.min, u8.abs
// u16.max, u16.min, u16.abs
// u32.max, u32.min, u32.abs
// u64.max, u64.min, u64.abs
// f32.max, f32.min, f32.abs
// f64.max, f64.min, f64.abs

// Usage examples:
val maxInt = i32.max(10, 20)          // 20
val minFloat = f64.min(3.14, 2.71)    // 2.71
val absValue = i16.abs(-42)           // 42
```

##### Power Functions

Power functions are available for numeric types:

```stria
// Integer power functions
fun i32.pow(base: i32, exponent: i32): i32 {
    base ** exponent
}

// Floating-point power functions
fun f64.pow(base: f64, exponent: f64): f64 {
    base ** exponent
}

// Similar functions exist for all other numeric types

// Usage examples:
val intPower = i32.pow(2, 3)          // 8
val floatPower = f64.pow(2.0, 3.0)    // 8.0
val mixedPower = f64.pow(2.0, 3.0)    // 8.0
```

##### Floating-Point Only Functions

The following functions are only available for floating-point types (`f32`, `f64`):

```stria
// Square root function
fun f64.sqrt(value: f64): f64 {
    if (value < 0) error("Cannot compute square root of negative number")
    value ** 0.5
}

fun f32.sqrt(value: f32): f32 {
    if (value < 0) error("Cannot compute square root of negative number")
    value ** 0.5
}

// Rounding functions (floating-point input, integer output)
fun f64.round(value: f64): i32 {
    if (value >= 0) (value + 0.5) as i32 else (value - 0.5) as i32
}

fun f64.floor(value: f64): i32 {
    if (value >= 0) value as i32 else (value as i32) - 1
}

fun f64.ceil(value: f64): i32 {
    if (value >= 0) (value as i32) + 1 else value as i32
}

fun f32.round(value: f32): i32 {
    if (value >= 0) (value + 0.5) as i32 else (value - 0.5) as i32
}

fun f32.floor(value: f32): i32 {
    if (value >= 0) value as i32 else (value as i32) - 1
}

fun f32.ceil(value: f32): i32 {
    if (value >= 0) (value as i32) + 1 else value as i32
}

// Angle conversion functions
fun f64.deg(radians: f64): f64 {
    radians * (180.0 / pi)
}

fun f64.rad(degrees: f64): f64 {
    degrees * (pi / 180.0)
}

fun f32.deg(radians: f32): f32 {
    radians * (180.0f32 / pi_f32)
}

fun f32.rad(degrees: f32): f32 {
    degrees * (pi_f32 / 180.0f32)
}

// Trigonometric functions
fun f64.sin(angle: f64): f64 {
    // Implementation provided by runtime
}

fun f64.cos(angle: f64): f64 {
    // Implementation provided by runtime
}

fun f64.tan(angle: f64): f64 {
    // Implementation provided by runtime
}

fun f64.asin(value: f64): f64 {
    // Implementation provided by runtime
}

fun f64.acos(value: f64): f64 {
    // Implementation provided by runtime
}

fun f64.atan(value: f64): f64 {
    // Implementation provided by runtime
}

fun f64.atan2(y: f64, x: f64): f64 {
    // Implementation provided by runtime
}

// f32 versions of trigonometric functions
fun f32.sin(angle: f32): f32 {
    // Implementation provided by runtime
}

fun f32.cos(angle: f32): f32 {
    // Implementation provided by runtime
}

fun f32.tan(angle: f32): f32 {
    // Implementation provided by runtime
}

fun f32.asin(value: f32): f32 {
    // Implementation provided by runtime
}

fun f32.acos(value: f32): f32 {
    // Implementation provided by runtime
}

fun f32.atan(value: f32): f32 {
    // Implementation provided by runtime
}

fun f32.atan2(y: f32, x: f32): f32 {
    // Implementation provided by runtime
}

// Hyperbolic functions
fun f64.sinh(value: f64): f64 {
    (f64.exp(value) - f64.exp(-value)) / 2
}

fun f64.cosh(value: f64): f64 {
    (f64.exp(value) + f64.exp(-value)) / 2
}

fun f64.tanh(value: f64): f64 {
    f64.sinh(value) / f64.cosh(value)
}

fun f32.sinh(value: f32): f32 {
    (f32.exp(value) - f32.exp(-value)) / 2
}

fun f32.cosh(value: f32): f32 {
    (f32.exp(value) + f32.exp(-value)) / 2
}

fun f32.tanh(value: f32): f32 {
    f32.sinh(value) / f32.cosh(value)
}

// Exponential and logarithmic functions
fun f64.exp(value: f64): f64 {
    // Implementation provided by runtime
}

fun f64.log(value: f64): f64 {
    if (value <= 0) error("Cannot compute logarithm of non-positive number")
    // Implementation provided by runtime
}

fun f64.log10(value: f64): f64 {
    if (value <= 0) error("Cannot compute logarithm base 10 of non-positive number")
    // Implementation provided by runtime
}

fun f32.exp(value: f32): f32 {
    // Implementation provided by runtime
}

fun f32.log(value: f32): f32 {
    if (value <= 0) error("Cannot compute logarithm of non-positive number")
    // Implementation provided by runtime
}

fun f32.log10(value: f32): f32 {
    if (value <= 0) error("Cannot compute logarithm base 10 of non-positive number")
    // Implementation provided by runtime
}

// Floating-point manipulation functions
fun f64.ldexp(value: f64, exponent: i32): f64 {
    value * (2 ** exponent)
}

fun f64.frexp(value: f64): (f64, i32) {
    if (value == 0.0) (0.0, 0)
    else {
        val exponent = f64.floor(f64.log(f64.abs(value)) / f64.log(2.0))
        val mantissa = value / (2 ** exponent)
        (mantissa, exponent)
    }
}

fun f64.modf(value: f64): (f64, f64) {
    val integerPart = if (value >= 0) f64.floor(value) as f64 else f64.ceil(value) as f64
    val fractionalPart = value - integerPart
    (fractionalPart, integerPart)
}

fun f64.fmod(x: f64, y: f64): f64 {
    if (y == 0) error("Division by zero")
    x - y * f64.floor(x / y)
}

fun f32.ldexp(value: f32, exponent: i32): f32 {
    value * (2 ** exponent)
}

fun f32.frexp(value: f32): (f32, i32) {
    if (value == 0.0f32) (0.0f32, 0)
    else {
        val exponent = f32.floor(f32.log(f32.abs(value)) / f32.log(2.0f32))
        val mantissa = value / (2 ** exponent)
        (mantissa, exponent)
    }
}

fun f32.modf(value: f32): (f32, f32) {
    val integerPart = if (value >= 0) f32.floor(value) as f32 else f32.ceil(value) as f32
    val fractionalPart = value - integerPart
    (fractionalPart, integerPart)
}

fun f32.fmod(x: f32, y: f32): f32 {
    if (y == 0) error("Division by zero")
    x - y * f32.floor(x / y)
}
```

##### Usage Examples

```stria
// Functions available for all numeric types
val maxInt = i32.max(10, 20)          // 20
val minFloat = f32.min(3.14f32, 2.71f32) // 2.71f32
val absNegative = i64.abs(-42)        // 42i64

// Floating-point only functions
val squareRoot = f64.sqrt(16.0)       // 4.0
val roundedValue = f64.round(3.14)    // 3i32
val sine = f64.sin(pi / 2)            // 1.0
val logarithm = f64.log(10.0)         // 2.302585092994046

// Type-specific usage
val intMax = i16.max(100, 200)        // 200i16
val floatMax = f32.max(1.5f32, 2.5f32) // 2.5f32
val angle = f64.deg(pi / 4)           // 45.0
val f32Sine = f32.sin(pi_f32 / 2)     // 1.0f32
```

#### Type Conversion Functions

```stria
// Type conversion functions (cast operations)
fun i32(value: any): i32 {
    // Converts value to i32 type
}

fun f64(value: any): f64 {
    // Converts value to f64 type
}

fun string(value: any): string {
    // Converts value to string type
}

fun bool(value: any): bool {
    // Converts value to bool type
}
```

### Built-in Methods

The following methods are available on primitive types without import:

#### String Methods

```stria
// String functions
val upper = str.toUpperCase()
val lower = str.toLowerCase()
val length = str.length()
```

#### List Methods

The `List<T>` type provides comprehensive methods combining the best of Rust's `Vec<T>` and Kotlin's `MutableList<T>`:

##### Mutation Methods (Available During Construction)

```stria
val items = ['a', 'b', 'c']

// Add elements
items.push('d')                     // Add to end: ['a', 'b', 'c', 'd']
items.insert(1, 'x')               // Insert at index: ['a', 'x', 'b', 'c', 'd']

// Remove elements
val last = items.pop()             // Remove and return last: 'd'
val removed = items.removeAt(1)    // Remove at index: removes 'x'
val success = items.remove('b')    // Remove by value: true if found
items.clear()                      // Remove all elements: []

// Modify elements
items[0] = 'A'                     // Set by index
items.set(0, 'A')                  // Alias for index assignment
```

##### Access Methods

```stria
val numbers = [1, 2, 3, 4, 5]

// Basic access
val size = numbers.size()          // 5
val length = numbers.length()      // Alias for size(): 5
val isEmpty = numbers.isEmpty()    // false
val isNotEmpty = numbers.isNotEmpty() // true

// Element access
val first = numbers.first()        // 1
val last = numbers.last()          // 5
val element = numbers[2]           // 3 (index access)
val element = numbers.get(2)       // 3 (method access)

// Safe access
val firstOrNull = numbers.firstOrNull()  // 1 (or null if empty)
val lastOrNull = numbers.lastOrNull()    // 5 (or null if empty)
val getOrNull = numbers.getOrNull(10)    // null (index out of bounds)
val getOrElse = numbers.getOrElse(10, 0) // 0 (default value)
```

##### Search and Query Methods

```stria
val words = ['hello', 'world', 'test']

// Contains and search
val contains = words.contains('hello')     // true
val indexOf = words.indexOf('world')       // 1
val lastIndexOf = words.lastIndexOf('test') // 2

// Predicates
val hasLong = words.any { it.length() > 4 }     // true
val allLong = words.all { it.length() > 3 }     // true
val countLong = words.count { it.length() > 4 } // 2

// Find operations
val found = words.find { it.startsWith('w') }      // 'world'
val foundOrNull = words.findOrNull { it.length() > 10 } // null
val foundLast = words.findLast { it.length() > 3 }     // 'test'
```

##### Functional Methods

```stria
val numbers = [1, 2, 3, 4, 5]

// Transform operations
val doubled = numbers.map { it * 2 }           // [2, 4, 6, 8, 10]
val strings = numbers.map { it.toString() }    // ['1', '2', '3', '4', '5']
val filtered = numbers.filter { it > 3 }       // [4, 5]
val evens = numbers.filter { it % 2 == 0 }     // [2, 4]

// Reduce operations
val sum = numbers.reduce { acc, it -> acc + it }     // 15
val product = numbers.fold(1) { acc, it -> acc * it } // 120
val joined = numbers.joinToString(', ')              // '1, 2, 3, 4, 5'

// Side effects
numbers.forEach { print(it) }                 // Prints: 12345
numbers.forEachIndexed { index, value ->
    print(`${index}: ${value}`)               // Prints: 0: 1, 1: 2, ...
}

// Utility operations for configuration
val unique = numbers.distinct()                      // Remove duplicates
val uniqueBy = numbers.distinctBy { it % 3 }         // Remove by key: [1, 2, 3]
val sortedUnique = numbers.distinct().sorted()       // [1, 2, 3, 4, 5]
```

##### Slicing and Sublist Methods

```stria
val items = ['a', 'b', 'c', 'd', 'e']

// Range access
val slice = items[1..3]            // ['b', 'c'] (exclusive upper bound)
val inclusiveSlice = items[1..=3]  // ['b', 'c', 'd'] (inclusive)
val sublist = items.subList(1, 3)  // ['b', 'c'] (fromIndex, toIndex)

// Take and drop
val first3 = items.take(3)         // ['a', 'b', 'c']
val last3 = items.takeLast(3)      // ['c', 'd', 'e']
val skip2 = items.drop(2)          // ['c', 'd', 'e']
val skipLast2 = items.dropLast(2)  // ['a', 'b', 'c']
```

##### List Combination Methods

```stria
val list1 = [1, 2, 3]
val list2 = [4, 5, 6]
val single = [7]

// Concatenation
val combined = list1 + list2       // [1, 2, 3, 4, 5, 6]
val withSingle = list1 + single    // [1, 2, 3, 7]
val withElement = list1 + 4        // [1, 2, 3, 4]

// Spread operator
val spread = [...list1, ...list2]  // [1, 2, 3, 4, 5, 6]
val mixed = [0, ...list1, 10, ...list2] // [0, 1, 2, 3, 10, 4, 5, 6]

// Extend (mutable)
list1.extend(list2)                // list1 becomes [1, 2, 3, 4, 5, 6]
```

##### Sorting and Ordering Methods

```stria
val numbers = [3, 1, 4, 1, 5, 9, 2, 6]

// Sort operations (return new list)
val sorted = numbers.sorted()                    // [1, 1, 2, 3, 4, 5, 6, 9]
val reversed = numbers.reversed()               // [6, 2, 9, 5, 1, 4, 1, 3]
val sortedBy = numbers.sortedBy { -it }         // [9, 6, 5, 4, 3, 2, 1, 1]

// In-place sort operations (mutate original)
numbers.sort()                                  // numbers becomes [1, 1, 2, 3, 4, 5, 6, 9]
numbers.reverse()                              // numbers becomes [9, 6, 5, 4, 3, 2, 1, 1]
numbers.shuffle()                              // numbers becomes randomly shuffled
```

##### Utility Methods

```stria
val items = ['a', 'b', 'c', 'a', 'b']

// Distinct operations (useful for configuration)
val unique = items.distinct()                   // ['a', 'b', 'c']
val uniqueBy = items.distinctBy { it.length() } // Distinct by computed key
val grouped = items.groupBy { it }              // {'a': ['a', 'a'], 'b': ['b', 'b'], 'c': ['c']}

// Conversion
val array = items.toArray()                     // Convert to array
val string = items.toString()                   // "['a', 'b', 'c', 'a', 'b']"

// Validation
val isValid = items.isValidIndex(2)             // true
val isSorted = [1, 2, 3].isSorted()            // true
```

---

## Error Handling

### Error Function

Stria does not have explicit `throw` statements. Instead, the `error()` function is used to terminate execution and report errors. When `error()` is called, it immediately interrupts processing similar to a `throw` statement in other languages.

```stria
fun error(message: string): void {
    // Terminates execution with the given error message
    // This function never returns - it always throws an error
}
```

#### Error Function Usage

The `error()` function accepts a message parameter that will be displayed in the error output:

```stria
fun validate(value: i32) {
    if (value < 0) error('Value must be non-negative')
    if (value > 1000) error('Value must not exceed 1000')
}

fun validatePort(port: u16) {
    if (port < 1 || port > 65535) {
        error('Port must be between 1 and 65535')
    }
}
```

#### Error Display Format

When `error()` is called, the error message displays carets (`^`) under the entire `error()` call:

```
error[E001]: Value must be non-negative
  --> config.stria:3:9
   |
 3 |     if (value < 0) error('Value must be non-negative')
   |                    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ execution terminated here
   |
```

The caret underline (`^^^^^^^`) spans the entire `error()` function call, indicating where execution was terminated.

### Error Messages

Stria provides Rust-style error messages with clear descriptions and suggestions:

```
error[E001]: Type mismatch
  --> config.stria:5:10
   |
 5 |     port = "8080"
   |            ^^^^^^ expected u16, found string
   |
help: convert string to u16
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

    port: u16
}
```

---

## Future Features

### Planned Features

#### Type Constraints

```stria
type Port = u16(min: 1, max: 65535)
type Email = string(format: 'email')
type Password = string(minLength: 8, maxLength: 64)
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
