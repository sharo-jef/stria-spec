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
as
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
val numbers = [1, 2, 3, 4, 5]
val strings = ['a', 'b', 'c']
val mixed = [1, 'two', true]
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

- `List<T>`: Ordered collection of elements
- `Range<T>`: Range of values (e.g., `1..10`)

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
val range = 1..10           // Range from 1 to 10
val rangeWithStep = 1..10 step 2  // 1, 3, 5, 7, 9
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
infix fun i32.add(other: i32): i32 = this + other
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
fun validate(value: i32) {
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

#### Match Expressions

```stria
val result = match value {
    is string -> "String value"
    is i32 -> "Integer value"
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
