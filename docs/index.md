# Stria Language Specification

- Version: none
- Date: none

## Table of Contents

1. [Introduction](#introduction)
2. [Language Overview](#language-overview)
3. [Lexical Structure](#lexical-structure)
4. [Types and Values](#types-and-values)
5. [Expressions and Operators](#expressions-and-operators)
6. [Statements and Control Flow](#statements-and-control-flow)
7. [Scope and Lifetime](#scope-and-lifetime)
8. [Structs and Initialization](#structs-and-initialization)
9. [Functions and Methods](#functions-and-methods)
10. [Annotations](#annotations)
11. [Schema System](#schema-system)
12. [Standard Library](#standard-library)
13. [Error Handling](#error-handling)
14. [AST Structure](#ast-structure)
15. [Future Features](#future-features)

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

### Configuration Language Nature

Stria is designed as a **configuration description language** that compiles to simple data formats like JSON, YAML, or TOML. This fundamental design principle influences all language features:

**Compilation Target**

```stria
// schema.stria
schema struct {
    var conf: ServerConfig
}

struct ServerConfig {
    var host: string
    var port: u16
    var ssl: bool
}
```

```stria
// config.stria
#schema './schema.stria'

conf {
    host = "api.example.com"
    port = 443
    ssl = true
}
```

```json
// Compiles to JSON:
{
  "conf": {
    "host": "api.example.com",
    "port": 443,
    "ssl": true
  }
}
```

**Configuration-Oriented Design**

- **Data-focused**: Methods are used to set configuration values, not to perform operations
- **Declarative**: Describes what the configuration should be, not how to achieve it
- **Stateless**: No runtime state changes or side effects beyond configuration assembly
- **Serializable**: All constructs map to simple data structures

**Not a General-Purpose Language**
Unlike general-purpose programming languages, Stria does not support:

- Runtime I/O operations
- Database connections
- Network requests
- File system operations
- Mutable state management

Instead, Stria focuses on expressing configuration logic, validation, and data transformation that can be evaluated at compile time.

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
myVariable123
```

### Keywords

Reserved keywords in Stria:

```
struct    init      fun       val       var
if        else      true      false     null
this      get       repeated  use       private
schema    mixin     error     match     when
as        is
```

**Property Modifiers:**

- `val`: Makes property immutable (single assignment only)
- `var`: Makes property mutable (can be reassigned)
- `private`: Restricts property access to within the struct only

**Annotations:**

- `@NoSerialize`: Excludes property from serialization output

**Note**: All properties must be explicitly declared with either `val` or `var` modifier. The `private` modifier and `@NoSerialize` annotation can be combined with either `val` or `var`.

### Literals

#### String Literals

```stria
val simpleString = 'Hello, World!'
val doubleQuoted = "Hello, World!"
val templateString = `Hello, ${name}!`
```

##### Template Literals

Template literals use backticks (`` ` ``) and support expression interpolation with `${expression}` syntax:

```stria
val name = "Alice"
val age = 30
val greeting = `Hello, ${name}! You are ${age} years old.`

// Complex expressions
val x = 10
val y = 20
val result = `The sum of ${x} and ${y} is ${x + y}`

// Function calls
val config = DatabaseConfig {
    host = "localhost"
    port = 5432
}
val connectionString = `Connected to ${config.host}:${config.port}`
```

##### Template Literal Type Conversion

Values embedded in template literals are automatically converted to strings by implicitly calling their `toString()` getter:

```stria
val number = 42
val boolean = true
val point = Point {
    x = 10
    y = 20
}

val message = `Number: ${number}, Boolean: ${boolean}, Struct: ${point}`
// Equivalent to:
val message = `Number: ${number.toString()}, Boolean: ${boolean.toString()}, Struct: ${point.toString()}`
```

**Type Conversion Rules:**

- **Primitive types**: Use built-in string representations
- **Struct types**: Call the `toString()` getter (implicit or user-defined)
- **Optional types**: `null` values are converted to `"null"`
- **Collections**: Use their default string representation or custom `toString()` implementation

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
val numbers = [1, 2, 3, 4, 5]            // i32[]
val strings = ['a', 'b', 'c']            // string[]
val mixed = [1, 'two', true]             // (i32 | string | bool)[]
val empty: i32[] = []                // Empty list with explicit type
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

#### Implementation Requirements

**Platform-Independent Types**
All numeric types must be implemented consistently across all platforms and execution environments. The language specification defines exact bit widths and value ranges that must be preserved regardless of the underlying hardware or runtime environment.

**Emulation Requirement**
If the target platform or runtime environment does not natively support a specific data type (e.g., `i64` on a 32-bit system, or `f32` on a platform with only double-precision floating-point), the implementation must provide complete emulation to ensure:

- Exact bit width and precision as specified
- Identical value ranges across all platforms
- Consistent arithmetic behavior and overflow handling
- Deterministic results for all operations

**Accuracy Over Performance**
As a configuration description language, Stria prioritizes correctness and consistency over execution speed. Implementations should:

- Use arbitrary precision arithmetic if necessary to ensure accuracy
- Implement proper rounding and precision handling for floating-point operations
- Ensure deterministic behavior for all numeric operations
- Provide exact emulation even if it results in slower execution

**Cross-Platform Consistency**
The same Stria configuration must produce identical results on all supported platforms:

- Integer overflow behavior must be consistent (wrap-around for unsigned, two's complement for signed)
- Floating-point operations must follow IEEE 754 standards precisely
- String encoding and representation must be uniform
- Boolean values must have identical truth semantics

**Examples of Required Consistency**

```stria
// These operations must produce identical results on all platforms:
val maxU64 = 18446744073709551615u64    // Must be exactly this value
val overflow = 255u8 + 1u8              // Must wrap to 0u8
val precision = 1.0f32 / 3.0f32         // Must have consistent precision
val bigInt = 9223372036854775807i64     // Must not depend on native int size
```

This ensures that configuration files are truly portable and produce consistent deployment artifacts regardless of the build environment.

#### Collection Types

**Note on Type Signatures**: While Stria does not have type signatures in user code, built-in types like `List<T>` and `Range<T>` are documented with generic type parameters to explain their behavior. These type parameters exist only in the compiler's type system and documentation, not in user syntax.

- `List<T>`: Immutable ordered collection of elements
- `Range<T>`: Range of values with exclusive upper bound (e.g., `1..10` for integers, `1.0..10.0` for floats)

##### List Type

The `List` type represents an ordered collection of elements. For configuration management convenience, lists are **mutable during construction** and become **immutable after execution completes**.

**List Initialization**: Lists can only be initialized using literal syntax:

- Array literals: `[1, 2, 3]`
- Spread syntax: `[...a, ...b]`
- Empty lists: `[]`

**Type Notation**: In documentation and type contexts, list types are written as `T[]` (e.g., `string[]`, `i32[]`).

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

Stria provides static member access for struct properties using dot notation. This ensures type safety and enables compile-time validation, which aligns with Stria's design philosophy as a configuration description language.

```stria
struct DatabaseConfig {
    var host: string
    var port: u16
    var username: string
    var password?: string
}

val config = DatabaseConfig {
    host = 'localhost'
    port = 5432
    username = 'admin'
    password = 'secret123'
}

// Static member access using dot notation
val host = config.host        // string
val port = config.port        // u16
val username = config.username // string
val password = config.password // string?
```

#### Type Safety

Member access is statically typed and validated at compile time:

```stria
// Valid - accessing existing properties
val serverHost = config.host
val serverPort = config.port

// Invalid - compile-time error for non-existent properties
// val invalid = config.nonExistentProperty  // Error: Property 'nonExistentProperty' does not exist
```

#### Optional Property Access

When accessing optional properties, the result is an optional type:

```stria
val password = config.password   // Type: string?

// Safe access with null checking
if (config.password != null) {
    val safePassword = config.password  // Type: string (narrowed from string?)
}

// Using null coalescing operator
val passwordOrDefault = config.password ?: 'default_password'
```

#### Configuration Management Examples

Member access is commonly used in configuration scenarios:

```stria
struct ServerConfig {
    var host: string
    var port: u16
    var ssl: bool
    var maxConnections?: u32
}

val server = ServerConfig {
    host = 'api.example.com'
    port = 443
    ssl = true
    maxConnections = 1000
}

// Accessing configuration values
val endpoint = `${server.host}:${server.port}`
val protocol = if (server.ssl) 'https' else 'http'
val connectionLimit = server.maxConnections ?: 500

// Building derived configuration
val fullUrl = `${protocol}://${server.host}:${server.port}`
```

#### Design Rationale

**Static Analysis Support**
Member access uses static property names only, enabling:

- **Compile-time validation**: All property access is verified at compile time
- **Type safety**: Property types are known and enforced statically
- **IDE support**: Autocomplete, refactoring, and error detection work reliably
- **Static analysis**: Tools can analyze property usage without runtime execution

**Configuration Language Alignment**
This approach aligns with Stria's nature as a configuration description language:

- **Predictable structure**: Configuration structure is known at compile time
- **Schema validation**: Property access can be validated against schemas
- **Serialization compatibility**: Static structure maps cleanly to JSON/YAML output
- **Maintenance**: Configuration changes are trackable and refactorable

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

Range expressions are interpreted as `RangeExpression` at compile time and handled as `Range<T>` at runtime:

```stria
struct Range<i32> {
    val from: i32
    val to: i32
    val step: i32 = 1
}

struct Range<f64> {
    val from: f64
    val to: f64
    val step: f64 = 1.0
}
```

**Note**: The `Range<T>` type signature is used here for documentation purposes only. In actual Stria code, users interact with ranges through syntax like `1..10`, `1..=10`, `1 until 10`, and `1..10 step 2`.

#### Range Methods

Range types support the following infix functions:

```stria
// Step function for Range<i32>
infix fun Range<i32>.step(step: i32): Range<i32> {
    val {from, to} = this
    Range<i32> {
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

// Contains function for Range<i32>
infix fun i32.in(range: Range<i32>): bool {
    if (range.step > 0) {
        this >= range.from && this <= range.to && ((this - range.from) % range.step == 0)
    } else {
        this <= range.from && this >= range.to && ((range.from - this) % (-range.step) == 0)
    }
}

// Until function for integers (exclusive upper bound)
infix fun i32.until(to: i32): Range<i32> {
    if (to == this) error('to must not be equal to from')
    if (to < this) error('to must be greater than from')
    Range<i32> {
        from = this
        this.to = to - 1
        step = 1
    }
}

// Down to function for integers (descending range)
infix fun i32.downTo(to: i32): Range<i32> {
    if (to == this) error('to must not be equal to from')
    if (to > this) error('to must be less than from')
    Range<i32> {
        from = this
        this.to = to
        step = -1
    }
}

// Step function for Range<f64>
infix fun Range<f64>.step(step: f64): Range<f64> {
    val {from, to} = this
    Range<f64> {
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

// Contains function for Range<f64>
infix fun f64.in(range: Range<f64>): bool {
    if (range.step > 0) {
        this >= range.from && this <= range.to && ((this - range.from) % range.step == 0)
    } else {
        this <= range.from && this >= range.to && ((range.from - this) % (-range.step) == 0)
    }
}

// Until function for floats (exclusive upper bound)
infix fun f64.until(to: f64): Range<f64> {
    if (to == this) error('to must not be equal to from')
    if (to < this) error('to must be greater than from')
    Range<f64> {
        from = this
        this.to = to
        step = 1.0
    }
}

// Down to function for floats (descending range)
infix fun f64.downTo(to: f64): Range<f64> {
    if (to == this) error('to must not be equal to from')
    if (to > this) error('to must be less than from')
    Range<f64> {
        from = this
        this.to = to
        step = -1.0
    }
}
```

#### Range Usage Examples

```stria
// Basic ranges
val numbers = 1..10          // Range<i32> from 1 to 10 (exclusive upper bound)
val floats = 1.0..10.0       // Range<f64> from 1.0 to 10.0 (exclusive upper bound)
val inclusive = 1..=10       // Range<i32> from 1 to 10 (inclusive)

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

### Statement Separators

Stria uses newlines as the primary statement separator, but semicolons (`;`) can be used to write multiple statements on a single line:

```stria
// Standard single-statement-per-line format
val x = 10
val y = 20
val result = x + y

// Multiple statements on one line using semicolons
val x = 10; val y = 20; val result = x + y

// Mixed format (discouraged for readability)
val x = 10; val y = 20
val result = x + y
```

**Usage Guidelines:**

- Use semicolons only when you need to write multiple statements on the same line
- Prefer newlines for better readability in most cases
- Semicolons are optional at the end of lines (newlines are sufficient)

### Variable Declarations

```stria
val immutableValue = 42        // Immutable variable
var mutableValue = 'hello'     // Mutable variable
```

**Note**: `val` and `var` keywords also serve as property modifiers in struct definitions:

- **Variable context**: `val` creates immutable variables, `var` creates mutable variables
- **Property context**: `val` creates immutable properties (single assignment), `var` creates mutable properties (default behavior)

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

#### Exhaustiveness Checking

Match expressions in Stria are required to be **exhaustive**, meaning they must handle all possible values of the matched expression. The compiler performs static analysis to verify exhaustiveness at compile time.

##### Exhaustiveness for Primitive Types

For primitive types with finite value spaces, exhaustiveness checking ensures all values are covered:

```stria
// Boolean exhaustiveness
val result = match flag {
    true -> "enabled"
    false -> "disabled"
    // No else clause needed - all boolean values are covered
}

// Using else clause for boolean (also valid)
val result = match flag {
    true -> "enabled"
    else -> "disabled"  // Covers false
}
```

##### Exhaustiveness for Union Types

Union types require all type variants to be explicitly handled:

```stria
union Value = string | i32 | bool

// Exhaustive match - all union variants covered
val description = match value {
    is string -> `String: ${value}`
    is i32 -> `Integer: ${value}`
    is bool -> `Boolean: ${value}`
    // No else clause needed - all union variants are covered
}

// Compile-time error: non-exhaustive match
val incomplete = match value {
    is string -> `String: ${value}`
    is i32 -> `Integer: ${value}`
    // Error: Missing pattern for 'bool'
}

// Using else clause for union types (also valid)
val withElse = match value {
    is string -> `String: ${value}`
    is i32 -> `Integer: ${value}`
    else -> `Other: ${value}`  // Covers remaining variants (bool)
}
```

##### Exhaustiveness for Numeric Types

For numeric types with large value spaces, an `else` clause is typically required:

```stria
// Exhaustive with else clause
val category = match age {
    0..12 -> "Child"
    13..19 -> "Teen"
    20..64 -> "Adult"
    65..150 -> "Senior"
    else -> "Invalid age"  // Required to cover remaining values
}

// Compile-time error: non-exhaustive match
val incomplete = match score {
    90..100 -> "A"
    80..89 -> "B"
    // Error: Not all possible i32 values are covered
}
```

##### Exhaustiveness for Optional Types

Optional types require handling both the `null` case and the non-null case:

```stria
val result = match optionalValue {
    null -> "No value"
    is string -> `Value: ${optionalValue}`
    // Exhaustive - both null and non-null cases covered
}

// Using else for optional types
val result = match optionalValue {
    null -> "No value"
    else -> `Value: ${optionalValue}`  // Covers all non-null cases
}

// Compile-time error: non-exhaustive match
val incomplete = match optionalValue {
    is string -> `Value: ${optionalValue}`
    // Error: Missing pattern for 'null'
}
```

##### Exhaustiveness with Value Patterns

When using specific value patterns, exhaustiveness checking verifies all possible values:

```stria
// Exhaustive enum-like matching
union Status = "success" | "error" | "pending"

val message = match status {
    "success" -> "Operation completed"
    "error" -> "Operation failed"
    "pending" -> "Operation in progress"
    // Exhaustive - all string literal values covered
}

// Compile-time error: non-exhaustive match
val incomplete = match status {
    "success" -> "Operation completed"
    "error" -> "Operation failed"
    // Error: Missing pattern for 'pending'
}
```

##### Nested Pattern Exhaustiveness

For nested patterns, exhaustiveness checking applies recursively:

```stria
union Result = Success | Error

struct Success {
    val value: string
}

struct Error {
    val code: i32
    val message: string
}

// Exhaustive nested matching
val output = match result {
    is Success -> match result.value {
        "" -> "Empty success"
        else -> `Success: ${result.value}`
    }
    is Error -> match result.code {
        404 -> "Not found"
        500 -> "Server error"
        else -> `Error ${result.code}: ${result.message}`
    }
}
```

##### Compiler Error Messages

The compiler provides detailed error messages for non-exhaustive matches:

```stria
// Example error message:
val incomplete = match value {
    is string -> "String"
    is i32 -> "Integer"
}
// Error: Match expression is not exhaustive
// Missing patterns for: bool
// Help: Add a pattern for 'bool' or use an 'else' clause
```

##### Exhaustiveness Checking Rules

1. **Static Analysis**: Exhaustiveness is checked at compile time, not runtime
2. **All Paths Required**: Every possible value must have a matching pattern
3. **Union Type Coverage**: All variants of union types must be explicitly handled
4. **Null Safety**: Optional types must handle both null and non-null cases
5. **Value Pattern Coverage**: When using specific values, all possible values must be covered
6. **Else Clause**: Can be used as a catch-all to cover remaining cases
7. **Dead Code Detection**: Unreachable patterns (after exhaustive coverage) generate warnings
8. **Order Independence**: Exhaustiveness checking is independent of pattern order

##### Performance Considerations

Exhaustiveness checking is performed entirely at compile time:

- **No Runtime Overhead**: All checks are resolved during compilation
- **Compile-time Guarantee**: Runtime will never encounter unhandled cases
- **Optimization Opportunity**: Compilers can optimize match expressions knowing all cases are handled
- **Static Analysis Tools**: IDE support for exhaustiveness checking and pattern completion

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

## Scope and Lifetime

### Scope Rules

Stria follows lexical scoping rules similar to modern programming languages, with specific considerations for configuration management use cases.

#### Global Scope

Variables and functions declared at the file level have global scope within that file:

```stria
// Global variables
val globalConfig = "production"
var globalCounter = 0

// Global function
fun getEnvironment(): string {
    globalConfig
}

// Available throughout the file
struct Config {
    init {
        environment = getEnvironment()
        counter = globalCounter
    }
    var environment: string
    var counter: i32
}
```

#### Block Scope

Variables declared within blocks (enclosed by `{}`) have block scope:

```stria
val outerValue = 10

if (condition) {
    val innerValue = 20        // Block-scoped variable
    var mutableInner = 30      // Block-scoped mutable variable

    // Both outerValue and innerValue are accessible here
    val sum = outerValue + innerValue  // 30
}

// innerValue is not accessible here
// val invalid = innerValue  // Error: 'innerValue' is not in scope
```

#### Function Scope

Function parameters and local variables have function scope:

```stria
fun calculateSum(a: i32, b: i32): i32 {
    val localMultiplier = 2           // Function-scoped variable
    val intermediate = a * localMultiplier

    if (b > 0) {
        val blockLocal = intermediate + b  // Block-scoped within if
        blockLocal
    } else {
        intermediate
    }
}

// localMultiplier is not accessible here
// val invalid = localMultiplier  // Error: 'localMultiplier' is not in scope
```

#### Struct Scope

Struct members have struct scope and are accessible within method bodies:

```stria
struct Calculator {
    var result: i32

    init {
        result = 0
    }

    fun add(value: i32) {
        // Both 'result' and 'value' are accessible here
        result += value
    }

    fun multiply(value: i32) {
        // 'result' is accessible, but 'value' from add() is not
        result *= value
    }
}
```

#### Lambda Scope

Lambda expressions have their own scope and can capture variables from enclosing scopes:

```stria
val multiplier = 3

val numbers = [1, 2, 3, 4, 5]
val transformed = numbers.map { value ->
    val localFactor = 2                    // Lambda-scoped variable
    (value * multiplier) + localFactor     // Captures 'multiplier' from outer scope
}

// localFactor is not accessible here
// val invalid = localFactor  // Error: 'localFactor' is not in scope
```

### Variable Shadowing

Stria allows variable shadowing, where inner scopes can declare variables with the same name as outer scopes:

#### Basic Shadowing

```stria
val name = "global"

fun processName(): string {
    val name = "function"     // Shadows global 'name'

    if (true) {
        val name = "block"    // Shadows function 'name'
        name                  // Returns "block"
    }

    name                      // Returns "function"
}

val result = processName()    // "function"
val global = name            // "global"
```

#### Shadowing in Struct Context

```stria
val defaultPort = 8080

struct ServerConfig {
    init(port: u16) {
        val defaultPort = 3000        // Shadows global 'defaultPort'
        this.port = port ?: defaultPort  // Uses local 'defaultPort' (3000)
    }

    var port: u16
}

val config = ServerConfig(null)  // config.port = 3000
val global = defaultPort         // 8080
```

#### Shadowing Rules

1. **Inner scope variables shadow outer scope variables** with the same name
2. **Shadowing is allowed across different scope levels** (global ↁEfunction ↁEblock)
3. **No shadowing within the same scope level** - redeclaration in the same scope is an error
4. **Shadowed variables are temporarily hidden** but not destroyed
5. **Original variables become accessible again** when the shadowing scope ends

#### Shadowing Restrictions

```stria
fun example() {
    val x = 10
    // val x = 20  // Error: Cannot redeclare 'x' in the same scope

    if (true) {
        val x = 30  // OK: Shadows the outer 'x'
        val y = 40
        // val y = 50  // Error: Cannot redeclare 'y' in the same scope
    }
}
```

### Variable Lifetime

#### Immutable Variables (`val`)

Immutable variables exist from declaration until the end of their scope:

```stria
fun example() {
    val immutable = "value"  // Lifetime starts here

    if (condition) {
        val blockScoped = immutable + "!"  // Lifetime starts here
        // blockScoped lifetime ends here
    }

    // immutable is still accessible
    val result = immutable
    // immutable lifetime ends here
}
```

#### Mutable Variables (`var`)

Mutable variables follow the same lifetime rules as immutable variables:

```stria
fun counter() {
    var count = 0        // Lifetime starts here

    for (i in 1..10) {
        var temp = i * 2  // Lifetime starts here (each iteration)
        count += temp
        // temp lifetime ends here (each iteration)
    }

    count               // count is still accessible
    // count lifetime ends here
}
```

#### Captured Variables in Lambdas

Variables captured by lambdas extend their lifetime:

```stria
fun createMultiplier(factor: i32): (i32) -> i32 {
    val multiplier = factor  // Lifetime extended by lambda capture

    { value ->
        value * multiplier   // 'multiplier' is captured and kept alive
    }
    // multiplier's lifetime is extended beyond this point
}

val doubler = createMultiplier(2)
val result = doubler(5)  // multiplier is still accessible here
```

### Struct Member Lifetime

#### Property Lifetime

Struct properties exist from struct instantiation until the struct instance is no longer referenced:

```stria
struct Config {
    init {
        timeout = 30      // Property lifetime starts here
    }

    var timeout: i32
}

val config = Config()     // config.timeout lifetime starts
// config.timeout lifetime continues until config is no longer referenced
```

#### Method Lifetime

Methods exist for the lifetime of the struct instance:

```stria
struct Calculator {
    fun add(value: i32) {
        result += value   // Method can access properties throughout struct lifetime
    }

    var result: i32
}

val calc = Calculator()   // calc.add() becomes available
calc.add(10)             // Method is accessible
// calc.add() lifetime ends when calc is no longer referenced
```

### Configuration-Specific Lifetime Considerations

#### Build-Time vs Runtime Lifetime

As a configuration description language, Stria has distinct lifetime phases:

```stria
// Schema definition phase
struct DatabaseConfig {
    init {
        // Build-time initialization
        connectionString = buildConnectionString()
    }

    fun buildConnectionString(): string {
        // This method executes during build-time
        `${host}:${port}/${database}`
    }

    var host: string
    var port: u16
    var database: string
    var connectionString: string
}

// Configuration instantiation phase
val config = DatabaseConfig {
    host = "localhost"
    port = 5432
    database = "myapp"
}

// Runtime phase (after compilation)
// The compiled JSON contains the final values:
// {
//   "host": "localhost",
//   "port": 5432,
//   "database": "myapp",
//   "connectionString": "localhost:5432/myapp"
// }
```

#### Immutable Runtime Values

After compilation, all configuration values become immutable:

```stria
// During build-time, these are mutable for construction
var servers = List<string>()
servers.push("web1.example.com")
servers.push("web2.example.com")

// After compilation, the resulting JSON is immutable:
// {
//   "servers": ["web1.example.com", "web2.example.com"]
// }
```

### Scope Resolution Order

When resolving variable names, Stria follows this precedence order:

1. **Current block scope** - Variables declared in the current block
2. **Enclosing block scopes** - Variables from outer blocks (innermost to outermost)
3. **Function scope** - Function parameters and local variables
4. **Struct scope** - Struct properties and methods (when inside a struct)
5. **File scope** - Global variables and functions
6. **Import scope** - Imported functions and types

#### Example of Scope Resolution

```stria
val global = "global"

fun example() {
    val function = "function"

    if (true) {
        val block = "block"

        // Resolution order for variable lookup:
        // 1. block scope: 'block' -> "block"
        // 2. function scope: 'function' -> "function"
        // 3. global scope: 'global' -> "global"

        val values = [block, function, global]  // ["block", "function", "global"]
    }
}
```

### Best Practices for Scope Management

#### Minimize Scope Pollution

```stria
// Good: Limit variable scope to where it's needed
fun processConfig() {
    val config = loadConfig()

    if (config.isValid()) {
        val processedData = processData(config)  // Limited to this block
        return processedData
    }

    return null
}

// Avoid: Unnecessarily wide scope
fun processConfigBad() {
    val config = loadConfig()
    val processedData = null  // Declared too early

    if (config.isValid()) {
        processedData = processData(config)
    }

    return processedData
}
```

#### Use Descriptive Names to Avoid Shadowing Confusion

```stria
// Good: Clear naming prevents confusion
val globalTimeout = 30

struct ServerConfig {
    init(requestTimeout: u32) {
        this.timeout = requestTimeout ?: globalTimeout
    }

    var timeout: u32
}

// Avoid: Confusing shadowing
val timeout = 30

struct ServerConfig {
    init(timeout: u32) {           // Shadows global 'timeout'
        this.timeout = timeout ?: timeout  // Confusing!
    }

    var timeout: u32
}
```

#### Prefer `val` Over `var` for Immutability

```stria
// Good: Immutable by default
fun calculateTotal(items: List<Item>): f64 {
    val subtotal = items.map { it.price }.sum()
    val tax = subtotal * 0.08
    val total = subtotal + tax

    total
}

// Avoid: Unnecessary mutability
fun calculateTotalBad(items: List<Item>): f64 {
    var subtotal = items.map { it.price }.sum()
    var tax = subtotal * 0.08
    var total = subtotal + tax

    total
}
```

---

## Structs and Initialization

### Basic Struct Definition

```stria
struct Person {
    var name: string
    var age: u8
    var email?: string  // Optional field
}
```

### Implicit toString() Getter

All structs automatically have an implicit `toString()` getter that provides a default string representation. This getter can be overridden by explicitly defining a custom `toString()` getter:

```stria
struct Point {
    val x: i32
    val y: i32
}

val point = Point { x = 10, y = 20 }
val defaultString = point.toString()  // Uses implicit toString()

// Custom toString() implementation
struct Person {
    var name: string
    var age: u8

    get toString(): string {
        `${name} (${age} years old)`
    }
}

val person = Person { name = "Alice", age = 30 }
val customString = person.toString()  // "Alice (30 years old)"
```

#### Template Literal Integration

The `toString()` getter is automatically invoked when structs are used in template literals:

```stria
struct ServerConfig {
    var host: string
    var port: u16

    get toString(): string {
        `${host}:${port}`
    }
}

val config = ServerConfig { host = "localhost", port = 8080 }
val message = `Server running on ${config}`  // "Server running on localhost:8080"
```

#### Default toString() Behavior

When no custom `toString()` getter is defined, the implicit implementation provides a default representation:

```stria
struct DatabaseConfig {
    var host: string
    var port: u16
    var username: string
}

val config = DatabaseConfig {
    host = "localhost"
    port = 5432
    username = "admin"
}

val defaultRepresentation = config.toString()
// Default output format: "DatabaseConfig { host: localhost, port: 5432, username: admin }"
```

**Default toString() Features:**

- **Struct name**: Includes the struct type name
- **Property listing**: Shows all properties and their values
- **Nested structs**: Recursively calls `toString()` on nested struct properties
- **Optional properties**: Shows `null` for unset optional properties
- **Consistent formatting**: Uses a predictable format for debugging and logging

### Member Name Constraints

**Same-Name Member Prohibition**
Structs cannot have properties and methods with the same name. This constraint ensures clarity and prevents ambiguity in member access:

```stria
struct Example {
    // Valid: Different names for property and method
    val value: i32

    fun getValue(): i32 {
        value
    }

    fun setValue(newValue: i32) {
        value = newValue
    }
}

struct Invalid {
    val value: i32

    // Error: Cannot define method with same name as property
    fun value(newValue: i32) {
        this.value = newValue
    }
}
```

**Design Rationale**
This constraint serves several purposes:

1. **Clarity**: Prevents confusion between property access and method calls
2. **Schema Validation**: Ensures clean mapping to configuration formats (JSON, YAML, TOML)
3. **Type Safety**: Eliminates ambiguity in member resolution
4. **Configuration Consistency**: Maintains predictable structure in configuration files

**Alternative Patterns**
Instead of same-name members, use descriptive method names:

```stria
struct ServerConfig {
    var host: string
    var port: u16

    // Good: Descriptive method names
    fun setHost(value: string) { host = value }
    fun setPort(value: u16) { port = value }
    fun getEndpoint(): string { `${host}:${port}` }
}
```

### Property Modifiers

Stria supports property modifiers and annotations that control mutability and serialization behavior:

#### Mutability Modifiers

All properties must be explicitly declared with either `val` or `var`:

##### `val` Properties (Immutable)

Properties marked with `val` can only be assigned once and become immutable thereafter:

```stria
struct ApplicationConfig {
    val appName: string        // Immutable property
    val version: string        // Immutable property
    var environment: string    // Mutable property
    var port: u16             // Mutable property

    init {
        appName = "MyApp"      // Valid: First assignment
        version = "1.0.0"      // Valid: First assignment
        environment = "development"
        port = 8080
    }

    fun updateEnvironment(env: string) {
        environment = env      // Valid: Mutable property
        port = 9000           // Valid: Mutable property
        // appName = "NewApp"  // Error: Cannot reassign immutable property
        // version = "2.0.0"   // Error: Cannot reassign immutable property
    }
}
```

##### `var` Properties (Mutable)

Properties marked with `var` can be reassigned multiple times:

```stria
struct ServerConfig {
    var host: string          // Mutable property
    var port: u16            // Mutable property
    var isActive: bool       // Mutable property

    init {
        host = "localhost"
        port = 8080
        isActive = true
    }

    fun updateServer(newHost: string, newPort: u16) {
        host = newHost        // Valid: Mutable property
        port = newPort        // Valid: Mutable property
        isActive = false      // Valid: Mutable property
    }
}
```

#### Access Modifiers

##### `private` Properties

Properties marked with `private` can only be accessed from within the struct itself:

```stria
struct UserAccount {
    var username: string           // Public property
    private var password: string   // Private property - only accessible within struct
    var email: string             // Public property

    init {
        username = "user123"
        password = "secret"        // Valid: Accessing private property within struct
        email = "user@example.com"
    }

    fun updatePassword(newPassword: string) {
        password = newPassword     // Valid: Accessing private property within struct
    }

    fun getHashedPassword(): string {
        hashPassword(password)     // Valid: Using private property within struct
    }
}

val account = UserAccount()
val user = account.username       // Valid: Accessing public property
val mail = account.email          // Valid: Accessing public property
// val secret = account.password  // Error: Cannot access private property
```

**Private Property Characteristics:**

- **Internal Access Only**: Accessible only within the struct's methods and initialization blocks
- **External Access Denied**: Cannot be accessed from outside the struct
- **Encapsulation**: Provides data hiding and encapsulation
- **Security**: Prevents direct access to sensitive or internal data
- **Still Serialized**: Private properties are included in serialization output by default (unless combined with `@NoSerialize`)

#### Serialization Annotations

##### `@NoSerialize` Annotation

Properties marked with `@NoSerialize` are excluded from serialization output but remain accessible within the struct. They serve as intermediate values used for computation:

```stria
struct DatabaseConfig {
    var host: string
    var port: u16
    var database: string
    @NoSerialize var rawPassword: string        // Input password, not serialized
    @NoSerialize var encryptionKey: string      // Internal key, not serialized
    var connectionString: string                // Computed from private values

    init {
        rawPassword = loadPassword()
        encryptionKey = generateKey()
        connectionString = buildSecureConnection()
    }

    fun buildSecureConnection(): string {
        // @NoSerialize properties are used in computation
        val hashedPassword = hash(rawPassword, encryptionKey)
        `${host}:${port}/${database}?auth=${hashedPassword}`
    }
}

val config = DatabaseConfig {
    host = "localhost"
    port = 5432
    database = "myapp"
}

// Serialized output excludes @NoSerialize properties:
// {
//   "host": "localhost",
//   "port": 5432,
//   "database": "myapp",
//   "connectionString": "localhost:5432/myapp?auth=abc123..."
// }
```

**@NoSerialize Use Cases:**

1. **Intermediate Calculations**: Hold temporary values used to compute final configuration
2. **Build-time Only Data**: Store information needed only during configuration construction
3. **Conditional Logic State**: Track internal state for complex initialization logic
4. **Performance Optimization**: Exclude large computed values from serialization

```stria
struct ApiConfig {
    var endpoint: string
    var timeout: u32
    @NoSerialize var rawApiKey: string          // Original API key
    @NoSerialize var keyVersion: string         // Key version info
    var hashedToken: string                     // Public token derived from private data

    init {
        rawApiKey = loadApiKey()
        keyVersion = detectKeyVersion(rawApiKey)
        hashedToken = generatePublicToken(rawApiKey, keyVersion)
        endpoint = selectEndpoint(keyVersion)
    }

    fun selectEndpoint(version: string): string {
        if (version == "v2") "https://api-v2.example.com" else "https://api.example.com"
    }
}

// Server configuration with build-time calculations
struct ServerConfig {
    var host: string
    var port: u16
    var maxConnections: u32
    @NoSerialize var cpuCores: u32              // Build-time system info
    @NoSerialize var memoryMB: u32              // Build-time system info
    @NoSerialize var loadFactor: f64            // Computed scaling factor

    init {
        cpuCores = detectCpuCores()
        memoryMB = detectMemoryMB()
        loadFactor = calculateOptimalLoad(cpuCores, memoryMB)
        maxConnections = (cpuCores * 100 * loadFactor) as u32
    }

    fun calculateOptimalLoad(cores: u32, memory: u32): f64 {
        // Complex calculation using @NoSerialize properties
        val coreWeight = cores as f64 * 0.3
        val memoryWeight = (memory / 1024) as f64 * 0.7
        (coreWeight + memoryWeight) / 100.0
    }
}
```

**@NoSerialize Property Characteristics:**

- **External Access**: Accessible from outside the struct (unless combined with `private`)
- **Internal Access**: Accessible within struct methods and initialization blocks
- **Serialization**: Excluded from JSON/YAML/TOML output
- **Schema Validation**: Not included in schema definitions
- **Build-time Processing**: Used for intermediate calculations and transformations
- **Security**: Prevents sensitive data from appearing in final configuration files

#### Combined Modifiers and Annotations

Modifiers and annotations can be combined to create properties with specific behavior:

```stria
struct SecurityConfig {
    val publicKey: string                       // Immutable, serialized
    @NoSerialize val privateKey: string         // Immutable, not serialized
    @NoSerialize var sessionCache: string?      // Mutable, not serialized
    var timeout: u32                            // Mutable, serialized
    private var internalState: string           // Mutable, serialized, access-controlled
    @NoSerialize private var secretData: string // Mutable, not serialized, access-controlled

    init {
        publicKey = loadPublicKey()
        privateKey = loadPrivateKey()
        sessionCache = null
        timeout = 3600
        internalState = "initialized"
        secretData = generateSecret()
    }

    fun generateToken(): string {
        // Use @NoSerialize properties for computation
        sessionCache = createSession(privateKey)
        `${publicKey}:${sessionCache}`
    }
}

// Serialized output includes only properties without @NoSerialize:
// {
//   "publicKey": "...",
//   "timeout": 3600,
//   "internalState": "initialized"
// }
```

#### Modifier and Annotation Rules

**Syntax Rules:**

- Modifiers (`val`/`var`) must appear before the property name
- Access modifiers (`private`) must appear before mutability modifiers
- Annotations (`@NoSerialize`) must appear before all modifiers
- All properties must have either `val` or `var` modifier

```stria
struct Example {
    @NoSerialize private val immutableSecret: string    // Correct order
    private val publicReadonly: string                  // Correct: access + mutability
    @NoSerialize var mutableSecret: string              // Correct: annotation + mutability
    var publicMutable: string                           // Correct: mutability only

    // Error examples:
    // val @NoSerialize invalid: string                 // Error: Wrong order
    // private @NoSerialize val invalid: string         // Error: Wrong order
    // secret: string                                   // Error: Missing val/var modifier
}
```

**Assignment Validation:**

- **`val` properties**: Must be assigned exactly once
- **`var` properties**: Can be assigned multiple times
- **`private` properties**: Only accessible within the struct
- **`@NoSerialize` properties**: Excluded from serialization output
- **`@NoSerialize` properties**: Excluded from serialization but otherwise behave normally
- **Assignment timing**: All required properties must be assigned before execution completes

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
    val connectionId: string              // Immutable, serialized
    var host: string                      // Mutable, serialized
    var port: u16                        // Mutable, serialized
    var username: string                 // Mutable, serialized
    var password?: string                // Optional, mutable, serialized
    @NoSerialize val encryptionKey: string // Immutable, not serialized
    @NoSerialize var connectionPool?: string // Mutable, not serialized
}

// Valid: All required properties assigned during initialization
val config1 = DatabaseConfig {
    connectionId = "db-conn-001"        // Immutable: can only be set once
    host = 'localhost'
    port = 5432
    username = 'admin'
    encryptionKey = 'secret-key-123'    // @NoSerialize immutable
}

// Also valid: Required properties assigned later during execution
val config2 = DatabaseConfig {}
config2.connectionId = "db-conn-002"   // First assignment - valid
config2.host = 'localhost'
config2.port = 5432
config2.username = 'admin'
config2.encryptionKey = 'secret-key-456'  // @NoSerialize immutable first assignment
// password remains null (implicitly initialized)
// connectionPool remains null (implicitly initialized)
// Execution completes successfully as all required properties are assigned

// Valid: Mutable property reassignment
config2.host = 'production-db.example.com'  // Valid: var property
config2.port = 5433                         // Valid: var property
config2.connectionPool = 'pool-connection'  // Valid: @NoSerialize var property

// Invalid: Immutable property reassignment
// config2.connectionId = "db-conn-003"     // Error: Cannot reassign val property
// config2.encryptionKey = "new-key"       // Error: Cannot reassign @NoSerialize val property

// Valid: Optional property assignment
val config3 = DatabaseConfig {
    connectionId = "db-conn-003"
    host = 'localhost'
    port = 5432
    username = 'admin'
    password = 'secret123'              // Optional property explicitly assigned
    encryptionKey = 'secret-key-789'
}

// Valid: Optional property accessed as null
val config4 = DatabaseConfig {
    connectionId = "db-conn-004"
    host = 'localhost'
    port = 5432
    username = 'admin'
    encryptionKey = 'secret-key-abc'
}
val hasPassword = config4.password != null  // false, password is null

// Serialization output for config4 (excludes @NoSerialize properties):
// {
//   "connectionId": "db-conn-004",
//   "host": "localhost",
//   "port": 5432,
//   "username": "admin",
//   "password": null
// }

// Invalid: Would cause runtime error
val config5 = DatabaseConfig {
    connectionId = "db-conn-005"
    host = 'localhost'
    port = 5432
    // username not assigned - runtime error at execution completion
    // encryptionKey not assigned - runtime error at execution completion
}
```

#### Property Modifier and Annotation Use Cases

**Configuration Management Examples:**

```stria
// Application Configuration
struct AppConfig {
    val appName: string                         // Immutable application identity
    val version: string                         // Immutable version info
    var environment: string                     // Mutable environment setting
    var debugMode: bool                        // Mutable debug flag
    @NoSerialize val buildId: string            // Internal build identifier, not serialized
    @NoSerialize var lastModified: string       // Internal timestamp, not serialized

    init {
        appName = "MyApplication"
        version = "1.0.0"
        environment = "production"
        debugMode = false
        buildId = generateBuildId()
        lastModified = currentTimestamp()
    }

    fun updateEnvironment(env: string) {
        environment = env
        lastModified = currentTimestamp()       // Update internal state
    }
}

// Server Configuration
struct ServerConfig {
    val serverId: string                        // Immutable server identity
    var host: string                           // Mutable host setting
    var port: u16                             // Mutable port setting
    @NoSerialize val sslCertPath: string       // SSL certificate path, not serialized
    @NoSerialize var requestCount: u32         // Internal metrics, not serialized

    init {
        serverId = generateServerId()
        host = "localhost"
        port = 8080
        sslCertPath = "/etc/ssl/certs/server.crt"
        requestCount = 0
    }

    fun incrementRequestCount() {
        requestCount += 1                       // Internal state tracking
    }
}

// API Configuration
struct ApiConfig {
    val apiKey: string                          // Immutable API key
    val apiVersion: string                      // Immutable API version
    var timeout: u32                           // Mutable timeout setting
    var retryCount: u8                         // Mutable retry setting
    @NoSerialize val secretKey: string          // Secret key, not serialized
    @NoSerialize var rateLimitRemaining: u32    // Rate limit tracking, not serialized

    init {
        apiKey = loadApiKey()
        apiVersion = "v1"
        timeout = 30000
        retryCount = 3
        secretKey = loadSecretKey()
        rateLimitRemaining = 1000
    }

    fun updateRateLimit(remaining: u32) {
        rateLimitRemaining = remaining          // Update internal state
    }
}
```

**Benefits of Property Modifiers and Annotations:**

1. **Security**: `@NoSerialize` properties keep sensitive data out of serialized output
2. **Immutability**: `val` properties prevent accidental modification of critical configuration
3. **Flexibility**: `var` properties allow runtime configuration updates
4. **Clean Output**: Serialized configuration contains only relevant external data
5. **Internal State**: `@NoSerialize` properties enable internal state management without exposing implementation details
6. **Explicit Declaration**: All properties must be explicitly declared as `val` or `var` for clarity

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
    val x: i32
    val y: i32
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
    val x: i32
    val y: i32
}
```

For parameterized initialization, the execution order is:

1. **Postfix Lambda**: Execution of the implicit postfix lambda block (if provided during instantiation)
2. **Initialization Block**: Execution of the `init` block defined in the struct

#### Primary Constructor

```stria
struct Point {
    init(this.x, this.y)
    val x: i32
    val y: i32
}
```

#### Primary Constructor with Initialization Block

Primary constructors can also include an initialization block for additional setup:

```stria
struct Point {
    init(this.x, this.y) {
        originDistance = math.sqrt(x ** 2 + y ** 2)
    }
    val x: i32
    val y: i32
    var originDistance: f64
}

// Configuration example
struct ServerConfig {
    init(this.host, this.port) {
        url = `http://${host}:${port}`
        isSecure = port == 443
    }
    var host: string
    var port: u16
    var url: string
    var isSecure: bool
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
    val value: i32
    var processed: i32
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

#### Design Rationale for Execution Order

The execution order (Parameter Assignment ↁEPostfix Lambda ↁEInit Block) is carefully designed to support flexible and intuitive configuration patterns:

**1. Parameter Assignment First**
Constructor parameters are assigned immediately to establish the basic structure. This ensures that:

- The instance has a valid initial state
- Subsequent steps can reference and modify these values
- Type safety is maintained throughout initialization

**2. Postfix Lambda Before Init Block**
The postfix lambda executes before the init block to enable a powerful configuration pattern:

```stria
struct ServerConfig {
    init(this.baseUrl) {
        // Step 3: Final validation and derived values
        if (!baseUrl.startsWith("http")) {
            error("Invalid URL format")
        }
        isSecure = baseUrl.startsWith("https")
        normalizedUrl = baseUrl.trimEnd('/')
    }

    var baseUrl: string
    var isSecure: bool
    var normalizedUrl: string
}

// The postfix lambda allows customization before final processing:
val config = ServerConfig("http://example.com") {
    // Step 2: Customize the base URL before validation
    baseUrl = baseUrl.replace("http://", "https://")
}
// Result: isSecure = true, normalizedUrl = "https://example.com"
```

**3. Benefits of This Order**

- **Customization before validation**: The postfix lambda can modify parameter values before the init block performs final validation and computations
- **Layered configuration**: Parameters provide defaults, postfix lambdas provide customization, and init blocks provide finalization
- **Predictable behavior**: Users can rely on this order when designing complex initialization logic

**4. Alternative Approaches Considered**
If the init block ran before the postfix lambda, it would limit flexibility:

- Validation would occur before user customization
- Derived values would be computed prematurely
- The pattern would be less intuitive for configuration use cases

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

    val major: u16
    val minor: u16
    val patch: u16
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
    val x: i32
    val y: i32
}

// Is actually equivalent to (shown as Kotlin-style pseudocode):
struct Point {
    init(action: Point.() -> Unit) {  // Kotlin-style pseudocode - not valid Stria syntax
        Point.action()  // Execute the lambda in the context of this instance
        x = 0
        y = 0
    }
    val x: i32
    val y: i32
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
    var command: string
}

struct TestStep {
    var script: string
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
    var name: string
}

struct IntegrationTest {
    var name: string
    var timeout: u32
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
    var mixinField: string = 'default'
}

struct MainStruct {
    mixin MixinExample
    var mainField: string
}
```

---

## Functions and Methods

### Function Declaration

```stria
fun buildConnectionString(host: string, port: u16, database: string): string {
    `${host}:${port}/${database}`
}

// Function with no return value (void return type is optional)
fun validateConfiguration(config: ServerConfig) {
    // Implementation here
}

// Explicit void return type (optional)
fun initializeDefaults(config: ServerConfig): void {
    // Implementation here
}
```

#### Return Types

Functions can have explicit return types or omit them:

- **Explicit return type**: `fun functionName(): ReturnType { ... }`
- **Omitted return type**: `fun functionName() { ... }` (implicitly returns `void`)
- **Void return type**: The `: void` annotation is optional for functions that don't return a value

#### Function Return Behavior

**Implicit Return Values**
Functions in Stria follow an expression-oriented approach where the last expression in a function body becomes the return value:

```stria
fun buildEndpoint(host: string, port: u16): string {
    val protocol = if (port == 443) "https" else "http"
    val endpoint = `${protocol}://${host}:${port}`
    endpoint  // This expression becomes the return value
}

fun getConfigPath(env: string): string {
    val basePath = "/etc/myapp"
    basePath + "/" + env + ".conf"  // This expression becomes the return value
}
```

**Explicit Return Statements**
Functions can use explicit `return` statements to return values early or for clarity:

```stria
fun selectDatabase(env: string): string {
    if (env == "production") return "prod_db"
    if (env == "staging") return "stage_db"
    return "dev_db"
}

fun validatePort(port: u16): string {
    if (port < 1) return "invalid"
    if (port > 65535) return "invalid"
    return "valid"
}
```

**Void Return Functions**
Functions that return `void` (or have no explicit return type) ignore the value of the final expression:

```stria
fun validateConfig(config: ServerConfig): void {
    // Implementation here
    42  // This value is ignored - function returns void
}

fun applyDefaults(config: ServerConfig) {  // Implicit void return
    // Implementation here
    "some string"  // This value is ignored - function returns void
}
```

**Mixed Return Patterns**
Functions can mix explicit returns with implicit returns:

```stria
fun getConfigValue(key: string, defaultValue: string): string {
    if (key == "database") return "localhost"  // Explicit return
    if (key == "port") return "5432"           // Explicit return
    defaultValue  // Implicit return (last expression)
}
```

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

    val value: i32
}
```

### Method Call Rules

#### Single Call Constraint

- Struct methods can be called only once by default
- Global/built-in functions can be called multiple times
- Getters can be called multiple times

```stria
struct Example {
    fun onceOnly() { /* ... */ }
    get property(): string { /* ... */ }
}
```

#### Design Rationale for Single Call Constraint

The "methods can only be called once" rule is a fundamental design principle in Stria that serves several critical purposes in configuration management:

**1. Preventing Duplicate Configuration**
This constraint prevents accidental duplicate configuration entries, which is the primary motivation for this rule:

```stria
struct Config {
    fun version(major: u16, minor: u16, patch: u16) {
        this.version = Version(major, minor, patch)
    }
    val version: Version
}

// This is the intended usage pattern:
val config = Config {
    version(0, 0, 1)
    // version(0, 0, 2) // This would be prevented by the single-call constraint
}
```

**2. Configuration Consistency**
Since Stria compiles to simple data formats like JSON, the single-call constraint ensures that configuration properties have exactly one value:

```stria
// schema.stria
schema {
    var config: ServerConfig
}

struct ServerConfig {
    var host: string
    var port: u16
}
```

```stria
// config.stria
#schema './schema.stria'

config {
    host = "localhost"
    port = 8080
    // host = "example.com" // Prevented - would cause ambiguity in final JSON
}
```

```json
// Compiles to clean JSON:
{
  "host": "localhost",
  "port": 8080
}
```

**3. Deterministic Configuration Evaluation**
This constraint ensures that configuration evaluation is deterministic and produces consistent results, which is essential for:

- Reproducible builds
- Predictable deployment configurations
- Reliable configuration validation

**4. Problems Without This Constraint**
Without the single-call constraint, configuration would become ambiguous:

```stria
// Problematic scenario without single-call constraint:
// schema.stria
schema {
    var config: DatabaseConfig
}

struct DatabaseConfig {
    var timeout: u32
}
```

```stria
// config.stria - This would be problematic without single-call constraint
#schema './schema.stria'

config {
    timeout = 30   // Should this be the final value?
    timeout = 60   // Or should this override the previous one?
}

// Resulting JSON would be ambiguous:
// Which timeout value should be used?
```

**5. Alignment with Configuration Languages**
This constraint aligns Stria with the nature of configuration languages:

- JSON objects have unique keys
- YAML mappings have unique keys
- Configuration files typically specify each setting once
- Multiple values for the same configuration key would be meaningless

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
    var name: string

    @description 'The user age in years'
    var age: u8

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
    var databaseUrl: string

    @name 'api_key'
    var apiKey: string

    @name 'max_connections'
    var maxConnections: u32
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
    var oldField: string

    var newField: string
}

@deprecated 'Use ModernConfig instead'
struct OldConfig {
    val value: string
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
    val major: u16
    val minor: u16
    val patch: u16

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
    val version: Version
}

struct Version {
    val major: u16
    val minor: u16
    val patch: u16

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
    val major: u16
    val minor: u16
    val patch: u16

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
    var host: string
    var port: u16
    var username: string
}

struct AppConfig {
    @flatten
    var database: DatabaseConfig

    var appName: string
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

Schema files define the structure and validation rules using the `schema` declaration. The `schema` keyword is used as a prefix for struct definitions, making schema definitions more consistent with regular struct syntax.

#### Schema Struct Syntax

There are three equivalent ways to define a schema struct:

**Full syntax with name:**

```stria
// schema.stria
schema struct AppConfig {
    var database: DatabaseConfig
    var server: ServerConfig
}

struct DatabaseConfig {
    var host: string
    var port: u16
}

struct ServerConfig {
    var host: string
    var port: u16
    var ssl: bool
}
```

**Abbreviated form 1 (struct name omitted):**

```stria
// schema.stria
schema struct {
    var database: DatabaseConfig
    var server: ServerConfig
}

struct DatabaseConfig {
    var host: string
    var port: u16
}

struct ServerConfig {
    var host: string
    var port: u16
    var ssl: bool
}
```

**Abbreviated form 2 (struct keyword omitted):**

```stria
// schema.stria
schema {
    var database: DatabaseConfig
    var server: ServerConfig
}

struct DatabaseConfig {
    var host: string
    var port: u16
}

struct ServerConfig {
    var host: string
    var port: u16
    var ssl: bool
}
```

#### Schema Semantics

- `schema struct Name {}` defines a schema with an optional name for documentation purposes
- `schema struct {}` defines a schema with the name omitted
- `schema {}` defines a schema with both the name and `struct` keyword omitted
- All three forms have identical semantics and behavior
- The schema struct is implicitly instantiated as a singleton when referenced by the `#schema` directive
- Only one struct definition with the `schema` keyword can be placed in a file

#### Schema Integration

The new schema definition syntax integrates seamlessly with existing Stria features:

- Schema structs can be used alongside regular struct definitions
- Schema structs support all existing validation features
- Schema structs follow the same grammar rules as regular structs

### Schema Reference

Configuration files reference schemas using the `#schema` directive. These files cannot contain schema declarations:

```stria
#schema './schema.stria'

database {
    host = 'localhost'
    port = 5432
}

server {
    host = 'api.example.com'
    port = 443
    ssl = true
}
```

### File Type Restrictions

Both schema files and configuration files use the `.stria` extension, but they serve different purposes with mutual exclusivity:

- **Schema Files**:

  - Must contain exactly one `schema struct` declaration (in any of the three forms)
  - Cannot use the `#schema` directive
  - Cannot contain configuration data
  - Define struct types and validation rules
  - Can contain additional struct definitions referenced by the schema

- **Configuration Files**:
  - Must use the `#schema` directive to reference a schema
  - Cannot contain `schema` declarations
  - Contain the actual configuration data
  - Must instantiate structs defined in the referenced schema
  - Top-level instantiation follows the definitions within the schema struct

### Schema Validation

- All configuration files must reference a schema
- Schema files cannot contain configuration data
- Runtime validation ensures type safety
- Compilation errors for schema violations
- Schema struct definitions are validated for consistency with regular struct syntax

---

## Standard Library

### Import System

The `use` statement is used to import standard library functions that are not built-in but are provided as standard functionality. These functions require explicit import before use.

```stria
use getEnv
use random
use print
use println
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
use println
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

use println

fun println(message: any) {
    // Prints message to standard output followed by a newline
    // Note: 'any' type does not exist in Stria's type system
    // This function exceptionally accepts any type through special compiler handling
}

// Usage examples:
print('Hello, ')
print('World!')      // Output: Hello, World!

println('Hello, World!')  // Output: Hello, World!\n
println(42)               // Output: 42\n
println(myStruct)         // Output: [struct representation]\n
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
    // Note: 'any' type does not exist in Stria's type system
    // This function exceptionally accepts any type through special compiler handling
}

fun f64(value: any): f64 {
    // Converts value to f64 type
    // Note: 'any' type does not exist in Stria's type system
    // This function exceptionally accepts any type through special compiler handling
}

fun string(value: any): string {
    // Converts value to string type
    // Note: 'any' type does not exist in Stria's type system
    // This function exceptionally accepts any type through special compiler handling
}

fun bool(value: any): bool {
    // Converts value to bool type
    // Note: 'any' type does not exist in Stria's type system
    // This function exceptionally accepts any type through special compiler handling
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

    var port: u16
}
```

---

## AST Structure

### Overview

The Abstract Syntax Tree (AST) is the intermediate representation of Stria source code used by parsers, compilers, and language tools. This section defines the structure and types of AST nodes for the Stria language.

### Common Node Structure

All AST nodes share a common base structure:

```typescript
interface BaseNode {
  var type: string;
  var loc: Location;
}

interface Location {
  var start: Position;
  var end: Position;
}

interface Position {
  var line: u32; // 1-based line number
  var column: u32; // 0-based column number (LSP compatible)
}
```

#### Position Information

**Line Numbers**: 1-based numbering (first line is line 1)
**Column Numbers**: 0-based numbering (first column is column 0)

This convention aligns with the Language Server Protocol (LSP) specification, which uses:

- 1-based line numbers
- 0-based column numbers

This ensures seamless integration with VSCode and other LSP-compatible editors without requiring position conversion.

### Node Type Definitions

#### Program Structure

```typescript
interface Program extends BaseNode {
  type: "Program";
  var body: Statement[];
}
```

#### Declarations

```typescript
interface SchemaDirective extends BaseNode {
  type: "SchemaDirective";
  var path: string;
}

interface UseStatement extends BaseNode {
  type: "UseStatement";
  var identifier: Identifier;
}

interface VariableDeclaration extends BaseNode {
  type: "VariableDeclaration";
  kind: "val" | "var";
  var name: Identifier;
  var typeRef: TypeExpression | null;
  val value: Expression;
  var annotations: Annotation[];
}

interface FunctionDeclaration extends BaseNode {
  type: "FunctionDeclaration";
  var isInfix: boolean;
  var receiverType: TypeExpression | null;
  var name: Identifier;
  var annotations: Annotation[];
  var parameters: Parameter[];
  var returnType: TypeExpression | null;
  var body: Statement[];
}

interface StructDeclaration extends BaseNode {
  type: "StructDeclaration";
  var name: Identifier;
  var annotations: Annotation[];
  var body: StructMember[];
}

interface UnionDeclaration extends BaseNode {
  type: "UnionDeclaration";
  var name: Identifier;
  var annotations: Annotation[];
  var types: TypeExpression[];
}

interface SchemaDeclaration extends BaseNode {
  type: "SchemaDeclaration";
  var annotations: Annotation[];
  var body: PropertyDeclaration[];
  // Note: Represents schema struct definitions using any of the three equivalent forms:
  // 1. schema struct Name { ... }
  // 2. schema struct { ... }
  // 3. schema { ... }
}

interface InitDeclaration extends BaseNode {
  type: "InitDeclaration";
  var annotations: Annotation[];
  var parameters: Parameter[];
  var body: Statement[];
}

interface MethodDeclaration extends BaseNode {
  type: "MethodDeclaration";
  var name: Identifier;
  var annotations: Annotation[];
  var isPrivate: boolean;
  var parameters: Parameter[];
  var returnType: TypeExpression | null;
  var body: Statement[];
}

interface GetterDeclaration extends BaseNode {
  type: "GetterDeclaration";
  var name: Identifier;
  var annotations: Annotation[];
  var returnType: TypeExpression | null;
  var body: Statement[];
}

interface PropertyDeclaration extends BaseNode {
  type: "PropertyDeclaration";
  var name: Identifier;
  var annotations: Annotation[];
  var isPrivate: boolean;
  var isOptional: boolean;
  var typeRef: TypeExpression;
  defaultValue?: Expression;
}

interface RepeatedDeclaration extends BaseNode {
  type: "RepeatedDeclaration";
  var name: Identifier;
  var annotations: Annotation[];
  var typeRef: TypeExpression;
  mapping?: MappingBlock;
}
```

#### Expressions

```typescript
interface CallExpression extends BaseNode {
  type: "CallExpression";
  var callee: Expression;
  var arguments: Expression[];
}

interface BinaryExpression extends BaseNode {
  type: "BinaryExpression";
  var operator: string;
  var left: Expression;
  var right: Expression;
}

interface LambdaExpression extends BaseNode {
  type: "LambdaExpression";
  var parameters: Parameter[];
  var body: Statement[];
}

interface IfExpression extends BaseNode {
  type: "IfExpression";
  var condition: Expression;
  var then: Statement[];
  else?: Statement[];
}

interface InfixCall extends BaseNode {
  type: "InfixCall";
  var functionName: string;
  var left: Expression;
  var right: Expression;
}

interface RangeExpression extends BaseNode {
  type: "RangeExpression";
  val from: Expression;
  val to: Expression;
  var inclusive: boolean;
}

interface TemplateLiteral extends BaseNode {
  type: "TemplateLiteral";
  parts: (TemplateStringPart | TemplateInterpolation)[];
}

interface TemplateStringPart extends BaseNode {
  type: "TemplateStringPart";
  val value: string;
}

interface TemplateInterpolation extends BaseNode {
  type: "TemplateInterpolation";
  var expression: Expression;
}

interface MemberAccessExpression extends BaseNode {
  type: "MemberAccessExpression";
  var object: Expression;
  var property: Identifier;
  var computed: boolean;
}

interface ThisExpression extends BaseNode {
  type: "ThisExpression";
}

interface NonNullAssertionExpression extends BaseNode {
  type: "NonNullAssertionExpression";
  var operand: Expression;
}

interface Identifier extends BaseNode {
  type: "Identifier";
  var name: string;
}
```

#### Literals

```typescript
interface StringLiteral extends BaseNode {
  type: "StringLiteral";
  val value: string;
  var raw: string;
}

interface IntegerLiteral extends BaseNode {
  type: "IntegerLiteral";
  val value: i64;
  var raw: string;
}

interface FloatLiteral extends BaseNode {
  type: "FloatLiteral";
  val value: f64;
  var raw: string;
}

interface BooleanLiteral extends BaseNode {
  type: "BooleanLiteral";
  val value: boolean;
  var raw: string;
}

interface NullLiteral extends BaseNode {
  type: "NullLiteral";
  var raw: string;
}

interface ListLiteral extends BaseNode {
  type: "ListLiteral";
  var elements: Expression[];
}
```

#### Statements

```typescript
interface AssignmentStatement extends BaseNode {
  type: "AssignmentStatement";
  operator: "=" | "+=" | "-=" | "*=" | "/=" | "%=";
  var left: Expression;
  var right: Expression;
}

interface ExpressionStatement extends BaseNode {
  type: "ExpressionStatement";
  var expression: Expression;
}

interface ReturnStatement extends BaseNode {
  type: "ReturnStatement";
  argument?: Expression;
}

interface BreakStatement extends BaseNode {
  type: "BreakStatement";
  label?: string;
}

interface ContinueStatement extends BaseNode {
  type: "ContinueStatement";
  label?: string;
}
```

#### Type Expressions

```typescript
interface TypeReference extends BaseNode {
  type: "TypeReference";
  var name: Identifier;
}

interface PrimitiveType extends BaseNode {
  type: "PrimitiveType";
  var name: string;
}

interface ArrayType extends BaseNode {
  type: "ArrayType";
  var elementType: TypeExpression;
  var dimensions: u32;
}

interface OptionalType extends BaseNode {
  type: "OptionalType";
  var baseType: TypeExpression;
}

interface UnionType extends BaseNode {
  type: "UnionType";
  var types: TypeExpression[];
}

interface FunctionType extends BaseNode {
  type: "FunctionType";
  var parameters: TypeExpression[];
  var returnType: TypeExpression;
}
```

#### Annotations and Metadata

```typescript
interface Annotation extends BaseNode {
  type: "Annotation";
  var name: Identifier;
  var arguments: Expression[];
}

interface Parameter extends BaseNode {
  type: "Parameter";
  var name: Identifier | MemberAccessExpression;
  typeRef?: TypeExpression;
}
```

#### Specialized Structures

```typescript
interface MappingBlock extends BaseNode {
  type: "MappingBlock";
  var entries: MappingEntry[];
}

interface MappingEntry extends BaseNode {
  type: "MappingEntry";
  val from: Identifier;
  val to: Identifier;
}
```

### Union Types

```typescript
type Expression =
  | CallExpression
  | BinaryExpression
  | LambdaExpression
  | IfExpression
  | InfixCall
  | RangeExpression
  | TemplateLiteral
  | MemberAccessExpression
  | ThisExpression
  | NonNullAssertionExpression
  | Identifier
  | StringLiteral
  | IntegerLiteral
  | FloatLiteral
  | BooleanLiteral
  | NullLiteral
  | ListLiteral;

type Statement =
  | AssignmentStatement
  | ExpressionStatement
  | ReturnStatement
  | BreakStatement
  | ContinueStatement
  | VariableDeclaration
  | FunctionDeclaration
  | StructDeclaration
  | UnionDeclaration
  | SchemaDeclaration
  | SchemaDirective
  | UseStatement;

type TypeExpression =
  | TypeReference
  | PrimitiveType
  | ArrayType
  | OptionalType
  | UnionType
  | FunctionType;

type StructMember =
  | InitDeclaration
  | MethodDeclaration
  | GetterDeclaration
  | PropertyDeclaration
  | RepeatedDeclaration;
```

### AST Usage Examples

#### Simple Variable Declaration

```stria
val message = "Hello, World!"
```

Generates:

```typescript
{
    type: 'Program',
    body: [
        {
            type: 'VariableDeclaration',
            kind: 'val',
            name: {
                type: 'Identifier',
                name: 'message'
            },
            var typeRef: null,
            value: {
                type: 'StringLiteral',
                value: 'Hello, World!',
                raw: '"Hello, World!"'
            },
            annotations: []
        }
    ]
}
```

#### Function Declaration

```stria
fun buildConnectionString(host: string, port: u16): string {
    `${host}:${port}`
}
```

Generates:

```typescript
{
    type: 'FunctionDeclaration',
    var isInfix: false,
    var receiverType: null,
    name: {
        type: 'Identifier',
        name: 'buildConnectionString'
    },
    annotations: [],
    parameters: [
        {
            type: 'Parameter',
            name: {
                type: 'Identifier',
                name: 'host'
            },
            typeRef: {
                type: 'PrimitiveType',
                name: 'string'
            }
        },
        {
            type: 'Parameter',
            name: {
                type: 'Identifier',
                name: 'port'
            },
            typeRef: {
                type: 'PrimitiveType',
                name: 'u16'
            }
        }
    ],
    returnType: {
        type: 'PrimitiveType',
        name: 'string'
    },
    body: [
        {
            type: 'ExpressionStatement',
            expression: {
                type: 'TemplateLiteral',
                parts: [
                    {
                        type: 'TemplateInterpolation',
                        expression: {
                            type: 'Identifier',
                            name: 'host'
                        }
                    },
                    {
                        type: 'TemplateStringPart',
                        value: ':'
                    },
                    {
                        type: 'TemplateInterpolation',
                        expression: {
                            type: 'Identifier',
                            name: 'port'
                        }
                    }
                ]
            }
        }
    ]
}
```

#### Struct Declaration

```stria
struct Point {
    val x: i32
    val y: i32
}
```

Generates:

```typescript
{
    type: 'StructDeclaration',
    name: {
        type: 'Identifier',
        name: 'Point'
    },
    annotations: [],
    body: [
        {
            type: 'PropertyDeclaration',
            name: {
                type: 'Identifier',
                name: 'x'
            },
            annotations: [],
            var isPrivate: false,
            var isOptional: false,
            typeRef: {
                type: 'PrimitiveType',
                name: 'i32'
            }
        },
        {
            type: 'PropertyDeclaration',
            name: {
                type: 'Identifier',
                name: 'y'
            },
            annotations: [],
            var isPrivate: false,
            var isOptional: false,
            typeRef: {
                type: 'PrimitiveType',
                name: 'i32'
            }
        }
    ]
}
```

### Implementation Guidelines

#### Parser Implementation

- **Position Tracking**: All nodes must include accurate location information for error reporting and tooling support
- **Type Safety**: Implement strict type checking during AST construction
- **Error Recovery**: Provide meaningful error messages with location information
- **Validation**: Ensure AST structure conforms to language semantics

#### Tool Integration

- **Language Servers**: Use AST for autocompletion, navigation, and refactoring
- **Code Formatters**: Preserve formatting preferences while maintaining AST structure
- **Static Analysis**: Traverse AST for linting, type checking, and code quality analysis
- **Documentation**: Generate API documentation from AST structure

#### Serialization

The AST structure is designed to be easily serializable to JSON or other formats for:

- **Debugging**: Inspecting parser output
- **Caching**: Storing parsed results for performance
- **Interoperability**: Sharing AST between different tools
- **Testing**: Comparing expected vs. actual parse results

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
