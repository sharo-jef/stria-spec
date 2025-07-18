# Proposal: Improved schema definition - 0002

- **Proposal ID**: 0002
- **Title**: Improved schema definition
- **Author**: sharo-jef
- **Status**: Draft
- **Category**: Schema System
- **Created**: 2025-07-18
- **Updated**: 2025-07-18

## Summary

This proposal aims to improve the schema definition capabilities of the Stria language by introducing new syntax and features that enhance the expressiveness and usability of schemas.

## Motivation

The current `schema {}` definitions may be treated as separate implementations from other struct definitions. This can lead to inconsistencies between struct definitions and schema definitions. This proposal aims to make schema definitions more intuitive and consistent.

## Detailed Design

We will improve schema definitions as follows:

- Use `schema` as a prefix for struct definitions like `schema struct Hoge {}`, rather than using `schema {}` as a standalone syntax.
- Only one struct definition with the `schema` keyword can be placed in a file.
- Since the `schema` keyword is merely a prefix, it will be treated the same as regular struct definitions thereafter.
- Structs with the `schema` keyword will be implicitly instantiated as singletons when referenced by the `#schema` directive.
  - On the config file side, top-level instantiation follows the definitions within the schema struct.
- The abbreviated forms are explained in the Syntax section below, but when using abbreviated form 2, it will look the same as the previous `schema {}` (although the grammar within the schema block will be completely identical to struct).

### Syntax

```stria
// schema.stria
schema struct Hoge {
  a: Fuga
}

struct Fuga {
   b: i32
   c: string
}
```

```
// schema.stria
// Abbreviated form 1 (struct name can be omitted when schema prefix is present)
schema struct {
  // fields
}

// Abbreviated form 2 (the `struct` keyword can also be omitted when schema prefix is present)
schema {
  // fields
}
```

```stria
// config.stria
#schema './schema.stria'
a {
   b = 1
   c = "example"
}
```

### Semantics

- `schema struct Hoge {}` defines a schema.
  - A name can be specified to align with existing struct definition syntax, but the name is optional and used only for documentation purposes.
- `schema struct {}` defines a schema similar to the above, but the name is omitted.
- `schema {}` defines a schema with both the name and `struct` keyword omitted. The meaning is the same as the above two forms.

### Integration

The new schema definition syntax integrates seamlessly with existing Stria features:

- It can be used alongside existing struct definitions.
- It supports all existing schema validation features.

## Examples

N/A

### Basic Usage

N/A

### Advanced Usage

N/A

### Configuration Context

N/A

## Rationale and Alternatives

N/A

### Why This Approach?

1. **Consistency**: Aligns schema definitions with existing struct definitions, making the language more intuitive.
2. **Simplicity**: Reduces the need for separate schema syntax, allowing users to define schemas in a familiar way.
3. **Flexibility**: Allows for both named and unnamed schema definitions, catering to different use cases.

### Alternative Designs Considered

N/A

## Implementation

To be determined

### Compiler Changes

To be determined

### Runtime Changes

To be determined

### Standard Library Changes

To be determined

### Migration Path

To be determined

## Compatibility

N/A

### Backward Compatibility

N/A

### Forward Compatibility

N/A

## Testing

N/A

### Unit Tests

N/A

### Integration Tests

N/A

### Performance Tests

N/A

## Documentation

- [x] Language specification
- [ ] Standard library documentation
- [x] Examples and tutorials
- [ ] Migration guide (if needed)

## Drawbacks

N/A

## Future Possibilities

N/A

## References

N/A

## Changelog

- **[2025-07-18]**: Initial proposal
