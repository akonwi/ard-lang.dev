---
title: Types
description: Overview of Ard's type system including built-in types, type unions, and nullable values.
---

## Built-in Types

Ard provides several fundamental types:

### Primitive Types

```ard
let text: Str = "Hello, World!"
let number: Int = 42
let decimal: Float = 3.14159
let flag: Bool = true
```

### Collection Types

```ard
// Lists (arrays)
let numbers: [Int] = [1, 2, 3, 4, 5]
let names: [Str] = ["Alice", "Bob", "Charlie"]

// Maps (dictionaries)
let scores: [Str:Int] = ["Alice": 95, "Bob": 87]
let config: [Str:Bool] = ["debug": true, "verbose": false]
```

## Type Inference

The compiler can infer types from context:

```ard
let count = 42           // Inferred as Int
let items = [1, 2, 3]    // Inferred as [Int]
let person = ["name": "Alice", "age": 30]  // Inferred as [Str:Str] or mixed
```

## Type Unions

Type unions allow a value to be one of several types:

```ard
type Printable = Str | Int
type Value = Int | Float | Str

let item: Printable = "Hello"
let data: Value = 42
```

### Working with Type Unions

Use match expressions to handle different types in a union:

```ard
type Content = Str | Int | Bool

fn describe(value: Content) Str {
  match value {
    Str => "Text: {it}"
    Int => "Number: {it.to_str()}"
    Bool => "Flag: {it.to_str()}"
  }
}

let items: [Content] = ["hello", 42, true]
for item in items {
  io::print(describe(item))
}
```

The `it` variable is automatically bound to the matched value.

## Nullable Types (Maybe)

Use `?` suffix to declare types that may or may not have a value:

```ard
use ard/maybe

mut maybe_name: Str? = maybe::none()
maybe_name = maybe::some("Alice")
```

### Working with Maybe Types

```ard
// Pattern matching
match maybe_name {
  name => "Hello, {name}!"
  _ => "Hello, stranger!"
}

// Checking presence
if maybe_name.is_some() {
  io::print("Name is present")
}

if maybe_name.is_none() {
  io::print("No name provided")
}

// Providing defaults
let name: Str = maybe_name.or("Anonymous")
```

### Maybe Type Methods

```ard
let maybe_value: Int? = maybe::some(42)

// Check if value is present
let has_value: Bool = maybe_value.is_some()
let is_empty: Bool = maybe_value.is_none()

// Get value or default
let value: Int = maybe_value.or(0)
let safe_value: Int = maybe_value.or_else(fn() Int { get_default() })
```

## Type Conversion

### Explicit Conversion

Use built-in methods for type conversion:

```ard
let num: Int = 42
let text: Str = num.to_str()        // "42"

let pi: Float = 3.14159
let rounded: Str = pi.to_str()      // "3.14159"

// String to number (returns Maybe types)
let maybe_num: Int? = "123".to_int()
let number: Int = maybe_num.or(0)   // 123 or 0 if conversion fails
```

### Safe Parsing

Conversion from strings returns Maybe types for safety:

```ard
let input = "not_a_number"
let maybe_num: Int? = input.to_int()

match maybe_num {
  value => io::print("Parsed: {value.to_str()}")
  _ => io::print("Invalid number format")
}
```

## Type Aliases

Create aliases for complex types:

```ard
type UserId = Int
type UserData = [Str:Str]
type ProcessResult = Result<Str, Str>

let user_id: UserId = 12345
let user_info: UserData = ["name": "Alice", "email": "alice@example.com"]
```

## Generic Syntax

While generics are a future feature, the syntax uses `$` prefix:

```ard
// Future syntax
type Container<$T> = {
  value: $T
}

fn identity<$T>(value: $T) $T {
  value
}
```

## Type Checking

The compiler performs strict type checking:

```ard
let number: Int = 42
let text: Str = "hello"

// This would be a compile error:
// let mixed = number + text  // Error: cannot add Int and Str

// Explicit conversion required:
let combined = number.to_str() + text  // "42hello"
```

## Best Practices

1. **Use type inference** when the type is obvious from context
2. **Prefer explicit types** for function parameters and return values
3. **Use Maybe types** instead of null values
4. **Use type unions** for values that can be one of several types
5. **Handle all cases** in match expressions for type unions and Maybe types