# Proposal: Enhanced String Interpolation - 0001

- **Proposal ID**: 0001
- **Title**: Enhanced String Interpolation with Format Specifiers
- **Author**: Example Author <example@example.com>
- **Status**: Draft
- **Category**: Language Features
- **Created**: 2025-01-18
- **Updated**: 2025-01-18

## Summary

This proposal introduces enhanced string interpolation capabilities with format specifiers, allowing developers to control the formatting of values within template literals more precisely.

## Motivation

Currently, Stria supports basic string interpolation using template literals:

```stria
val name = "Alice"
val age = 30
val message = `Hello, ${name}! You are ${age} years old.`
```

However, there's no way to control the formatting of interpolated values. This limitation becomes apparent when working with:

- Numeric precision (e.g., displaying 2 decimal places)
- Date/time formatting
- Padding and alignment
- Number bases (hex, binary, octal)

Configuration management often requires precise formatting for output generation, making this a valuable addition to the language.

## Detailed Design

### Syntax

The proposal extends the existing template literal syntax with format specifiers:

```stria
val formatted = `Value: ${expression:format}`
```

Where `:format` is an optional format specifier that controls how the value is rendered.

### Format Specifiers

#### Numeric Formatting

```stria
val price = 19.99
val formatted = `Price: $${price:.2f}`  // "Price: $19.99"

val count = 42
val hex = `Hex: ${count:x}`             // "Hex: 2a"
val binary = `Binary: ${count:b}`       // "Binary: 101010"
val padded = `Count: ${count:04d}`      // "Count: 0042"
```

#### String Formatting

```stria
val name = "Alice"
val left = `Name: ${name:<10}`          // "Name: Alice     "
val right = `Name: ${name:>10}`         // "Name:      Alice"
val center = `Name: ${name:^10}`        // "Name:   Alice   "
```

#### Boolean Formatting

```stria
val enabled = true
val yesno = `Enabled: ${enabled:yes/no}`    // "Enabled: yes"
val onoff = `Enabled: ${enabled:on/off}`    // "Enabled: on"
```

### Semantics

Format specifiers are processed at compile time and generate appropriate formatting code. The formatting is type-aware and validates specifier compatibility with the expression type.

### Integration

The enhanced interpolation works with:

- All existing template literal contexts
- Struct `toString()` methods
- Configuration output generation
- Error messages and logging

## Examples

### Basic Usage

```stria
val temperature = 23.456
val humidity = 67.8

val report = `Temperature: ${temperature:.1f}°C, Humidity: ${humidity:.0f}%`
// "Temperature: 23.5°C, Humidity: 68%"
```

### Advanced Usage

```stria
struct ServerStatus {
    name: string
    uptime: f64
    connections: u32

    get toString(): string {
        `Server ${name:<12} | Uptime: ${uptime:8.2f}h | Connections: ${connections:>6d}`
    }
}

val server = ServerStatus {
    name = "web-01"
    uptime = 123.45
    connections = 42
}

println(server)  // "Server web-01       | Uptime:   123.45h | Connections:     42"
```

### Configuration Context

```stria
struct DatabaseConfig {
    host: string
    port: u16
    timeout: f64

    get connectionString(): string {
        `${host}:${port:05d}?timeout=${timeout:.1f}s`
    }
}

val config = DatabaseConfig {
    host = "localhost"
    port = 5432
    timeout = 30.0
}

val connStr = config.connectionString  // "localhost:05432?timeout=30.0s"
```

## Rationale and Alternatives

### Why This Approach?

1. **Familiar syntax**: Similar to Python f-strings and C# string interpolation
2. **Type safety**: Format specifiers are validated at compile time
3. **Performance**: Formatting is optimized at compile time
4. **Readability**: Format is specified inline with the value

### Alternative Designs Considered

1. **Separate formatting functions**:

   - Pros: More explicit, no syntax changes
   - Cons: Verbose, breaks string flow
   - Rejected because: Reduces readability and requires learning many function names

2. **Printf-style formatting**:
   - Pros: Familiar to C/C++ developers
   - Cons: Not type-safe, error-prone
   - Rejected because: Doesn't fit Stria's type safety goals

## Implementation

### Compiler Changes

- Parse format specifiers in template literals
- Validate specifier compatibility with expression types
- Generate appropriate formatting code
- Optimize common formatting patterns

### Runtime Changes

- Add formatting functions for each supported type
- Implement efficient numeric formatting
- Support custom formatting for user-defined types

### Standard Library Changes

- Add format validation functions
- Extend numeric types with formatting methods
- Add string padding and alignment functions

### Migration Path

This change is fully backward compatible. Existing template literals continue to work unchanged.

## Compatibility

### Backward Compatibility

✅ **Fully backward compatible**. No existing code needs to change.

### Forward Compatibility

This feature provides a foundation for future enhancements like:

- Custom format specifiers for user-defined types
- Locale-aware formatting
- Advanced numeric formatting options

## Testing

### Unit Tests

- Format specifier parsing
- Type compatibility validation
- Formatting correctness for all types
- Edge cases (empty strings, null values, etc.)

### Integration Tests

- Template literals in various contexts
- Struct toString() methods
- Configuration output generation
- Error handling for invalid specifiers

### Performance Tests

- Formatting performance vs. manual string building
- Memory usage with complex formatting
- Compile-time optimization effectiveness

## Documentation

- [x] Language specification (template literals section)
- [x] Standard library documentation (formatting functions)
- [x] Examples and tutorials
- [ ] Migration guide (not needed - backward compatible)

## Drawbacks

- **Learning curve**: Users need to learn format specifier syntax
- **Complexity**: Adds parsing complexity to template literals
- **Compile time**: May increase compilation time for heavy formatting
- **Maintenance**: More code to maintain and test

## Future Possibilities

- **Custom formatters**: Allow user-defined types to specify custom format handlers
- **Locale support**: Add locale-aware formatting for numbers and dates
- **Advanced alignment**: Support for multi-line alignment and complex layouts
- **Performance optimizations**: Cache formatted strings for repeated use

## References

- [Python f-string formatting](https://docs.python.org/3/library/string.html#format-string-syntax)
- [C# string interpolation](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/tokens/interpolated)
- [Rust format! macro](https://doc.rust-lang.org/std/macro.format.html)

## Changelog

- **2025-01-18**: Initial proposal
