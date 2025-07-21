---
title: Error Handling
description: Learn about Ard's approach to error handling using Result types and the try keyword.
---

## No Exceptions

Ard does not have exceptions. Instead, errors are represented as values using the built-in `Result<$Val, $Err>` type. This approach makes error handling explicit and predictable.

## Result Type

The `Result` type represents either a successful value or an error:

```ard
// Function that might fail
fn divide(a: Int, b: Int) Result<Int, Str> {
  match b == 0 {
    true => Result::err("Cannot divide by zero")
    false => Result::ok(a / b)
  }
}
```

## Result Declaration Sugar

For convenience, `Result<T, E>` can be written using the sugar syntax `T!E`:

```ard
// These are equivalent:
fn divide_verbose(a: Int, b: Int) Result<Int, Str> { ... }
fn divide_concise(a: Int, b: Int) Int!Str { ... }
```

The `!` syntax is more concise and commonly used.

## Handling Results with Match

Use match expressions to handle both success and error cases:

```ard
fn safe_divide(x: Int, y: Int) {
  match divide(x, y) {
    ok(result) => io::print("Result: {result.to_str()}")
    err(error) => io::print("Error: {error}")
  }
}

// Usage
safe_divide(10, 2)  // "Result: 5"
safe_divide(10, 0)  // "Error: Cannot divide by zero"
```

## Ignoring Errors with `.or()`

To provide a default value when a result is an error:

```ard
let result = divide(10, 0).or(0)  // Returns 0 on error
io::print("Got: {result.to_str()}")  // "Got: 0"

let safe_result = divide(10, 2).or(999)  // Returns 5 (success value)
io::print("Got: {safe_result.to_str()}")  // "Got: 5"
```

## Error Propagation with `try`

The `try` keyword allows propagating errors to the calling function:

```ard
fn do_math(a: Int, b: Int) Int!Str {
  let quotient = try divide(a, b)  // If error, return early
  Result::ok(quotient + 10)        // Continue with success value
}

// Usage
match do_math(10, 2) {
  ok(value) => io::print("Final result: {value.to_str()}")  // "Final result: 15"
  err(error) => io::print("Math failed: {error}")
}

match do_math(10, 0) {
  ok(value) => io::print("Final result: {value.to_str()}")
  err(error) => io::print("Math failed: {error}")  // "Math failed: Cannot divide by zero"
}
```

**Important**: `try` can only be used within function bodies that return a Result type.

## Chaining Operations

Multiple operations that might fail can be chained:

```ard
fn complex_calculation(a: Int, b: Int, c: Int) Int!Str {
  let step1 = try divide(a, b)
  let step2 = try divide(step1, c)
  let step3 = try divide(step2, 2)
  Result::ok(step3 * 100)
}
```

If any step fails, the error propagates immediately.

## Custom Error Types

Create custom error types for more specific error handling:

```ard
enum MathError {
  division_by_zero,
  overflow,
  invalid_input
}

fn safe_divide(a: Int, b: Int) Int!MathError {
  match b {
    0 => Result::err(MathError::division_by_zero)
    _ => {
      // Check for potential overflow
      match would_overflow(a, b) {
        true => Result::err(MathError::overflow)
        false => Result::ok(a / b)
      }
    }
  }
}

// Handling custom errors
match safe_divide(100, 0) {
  ok(result) => io::print("Success: {result.to_str()}")
  err(error) => match error {
    MathError::division_by_zero => io::print("Cannot divide by zero")
    MathError::overflow => io::print("Result would overflow")
    MathError::invalid_input => io::print("Invalid input provided")
  }
}
```

## Multiple Error Types

Functions can return different error types using type unions:

```ard
type FileError = IoError | ParseError | ValidationError

fn process_file(path: Str) Str!FileError {
  let content = try read_file(path)      // Returns Str!IoError
  let data = try parse_content(content)  // Returns Data!ParseError
  let valid = try validate_data(data)    // Returns Data!ValidationError
  Result::ok(format_output(valid))
}
```

## Result Helper Functions

Common patterns for working with Results:

```ard
// Check if result is success without consuming it
let result = divide(10, 2)
if result.is_ok() {
  io::print("Operation succeeded")
}

if result.is_err() {
  io::print("Operation failed")
}

// Transform success values
fn double_if_success(result: Int!Str) Int!Str {
  match result {
    ok(value) => Result::ok(value * 2)
    err(error) => Result::err(error)
  }
}

// Convert errors
fn convert_error(result: Int!Str) Int!MathError {
  match result {
    ok(value) => Result::ok(value)
    err(_) => Result::err(MathError::division_by_zero)
  }
}
```

## Best Practices

### 1. Be Explicit About Errors

```ard
// Good: specific error type
fn parse_age(input: Str) Int!ParseError {
  // ...
}

// Less ideal: generic string error
fn parse_age(input: Str) Int!Str {
  // ...
}
```

### 2. Use `try` for Happy Path

```ard
// Good: focus on the success case
fn process_user_data(raw: Str) User!ProcessingError {
  let parsed = try parse_json(raw)
  let validated = try validate_user(parsed)
  let user = try create_user(validated)
  Result::ok(user)
}

// Verbose alternative with explicit matching
fn process_user_data_verbose(raw: Str) User!ProcessingError {
  match parse_json(raw) {
    ok(parsed) => match validate_user(parsed) {
      ok(validated) => match create_user(validated) {
        ok(user) => Result::ok(user)
        err(error) => Result::err(error)
      }
      err(error) => Result::err(error)
    }
    err(error) => Result::err(error)
  }
}
```

### 3. Provide Meaningful Error Messages

```ard
fn read_config_file(path: Str) Config!Str {
  match file_exists(path) {
    false => Result::err("Config file not found at: {path}")
    true => match read_file(path) {
      ok(content) => parse_config(content)
      err(_) => Result::err("Failed to read config file: {path}")
    }
  }
}
```

### 4. Use `.or()` for Simple Defaults

```ard
// Simple cases
let port = config.get_port().or(8080)
let timeout = settings.get_timeout().or(30)

// Complex cases: use explicit matching
match config.get_database_url() {
  ok(url) => connect_to_database(url)
  err(_) => {
    io::print("No database URL configured, using local SQLite")
    create_local_database()
  }
}
```