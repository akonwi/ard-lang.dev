---
title: Pattern Matching
description: Learn about Ard's powerful pattern matching with match expressions.
---

## Match Expressions

Match expressions provide powerful pattern matching capabilities in Ard. They are similar to switch statements but more expressive:

```ard
let result = match value {
  pattern1 => expression1
  pattern2 => expression2
  _ => default_expression
}
```

The `_` wildcard matches any value and serves as a catch-all pattern.

## Integer Patterns

### Specific Values

```ard
let message = match status_code {
  200 => "OK"
  404 => "Not Found"
  500 => "Server Error"
  _ => "Unknown Status"
}
```

### Range Patterns

Integer ranges are inclusive and use the `..` syntax:

```ard
let grade = match score {
  0..59 => "F"
  60..69 => "D"
  70..79 => "C"
  80..89 => "B"
  90..100 => "A"
  _ => "Invalid score"
}
```

### Mixed Patterns

Combine specific values and ranges:

```ard
let category = match age {
  0 => "newborn"
  1..12 => "child"
  13..19 => "teenager"
  21 => "legal drinking age"
  22..64 => "adult"
  65..120 => "senior"
  _ => "invalid age"
}
```

**Important**: When patterns overlap, the first match wins. Order patterns from most specific to least specific:

```ard
// Good: specific cases first
match number {
  42 => "the answer"     // Specific case
  40..50 => "forties"    // Range that includes 42
  _ => "other"
}

// Problematic: general case first
match number {
  40..50 => "forties"    // This catches 42
  42 => "the answer"     // This will never match
  _ => "other"
}
```

## Boolean Patterns

```ard
let access_level = match is_admin {
  true => "full access"
  false => "limited access"
}

// More complex boolean logic
let message = match logged_in && verified {
  true => "Welcome to your account"
  false => "Please log in and verify your account"
}
```

## String Patterns

```ard
let response = match command {
  "start" => "Starting the process..."
  "stop" => "Stopping the process..."
  "restart" => "Restarting the process..."
  "status" => get_status()
  _ => "Unknown command"
}
```

## Enum Patterns

```ard
enum Priority { low, medium, high, critical }

let urgency = match task_priority {
  Priority::low => "Can wait"
  Priority::medium => "Normal priority"
  Priority::high => "Important"
  Priority::critical => "Urgent!"
}
```

## Type Union Patterns

When matching on type unions, the matched value is bound to the variable `it`:

```ard
type Content = Str | Int | Bool

fn describe(value: Content) Str {
  match value {
    Str => "Text: '{it}'"
    Int => "Number: {it.to_str()}"
    Bool => "Boolean: {it.to_str()}"
  }
}

// Usage
let items: [Content] = ["hello", 42, true]
for item in items {
  io::print(describe(item))
}
// Output:
// Text: 'hello'
// Number: 42
// Boolean: true
```

## Maybe Type Patterns

```ard
use ard/maybe

let maybe_name: Str? = maybe::some("Alice")

let greeting = match maybe_name {
  name => "Hello, {name}!"      // Binds the value if present
  _ => "Hello, stranger!"       // Matches none case
}
```

## Result Type Patterns

```ard
fn divide(a: Int, b: Int) Result<Int, Str> {
  match b == 0 {
    true => Result::err("Division by zero")
    false => Result::ok(a / b)
  }
}

let result = divide(10, 2)
let message = match result {
  ok(value) => "Result: {value.to_str()}"
  err(error) => "Error: {error}"
}
```

## Complex Matching Examples

### State Machine

```ard
enum State { idle, running, paused, error }
enum Event { start, pause, resume, stop, fail }

fn handle_event(current_state: State, event: Event) State {
  match current_state {
    State::idle => match event {
      Event::start => State::running
      _ => State::idle
    }
    State::running => match event {
      Event::pause => State::paused
      Event::stop => State::idle
      Event::fail => State::error
      _ => State::running
    }
    State::paused => match event {
      Event::resume => State::running
      Event::stop => State::idle
      _ => State::paused
    }
    State::error => match event {
      Event::start => State::running
      _ => State::error
    }
  }
}
```

### HTTP Status Code Handling

```ard
fn handle_http_response(status: Int, body: Str) Str {
  match status {
    200..299 => "Success: {body}"
    300..399 => "Redirect: {body}"
    400 => "Bad Request"
    401 => "Unauthorized"
    403 => "Forbidden"
    404 => "Not Found"
    400..499 => "Client Error: {status.to_str()}"
    500..599 => "Server Error: {status.to_str()}"
    _ => "Unknown Status: {status.to_str()}"
  }
}
```

## Match Expression as Statement

Match expressions can be used as statements without returning a value:

```ard
match log_level {
  LogLevel::debug => io::print("DEBUG: {message}")
  LogLevel::info => io::print("INFO: {message}")
  LogLevel::warn => io::print("WARN: {message}")
  LogLevel::error => {
    io::print("ERROR: {message}")
    log_to_file(message)
  }
}
```

## Best Practices

1. **Handle all cases**: Always include a wildcard `_` pattern unless all possible values are explicitly handled
2. **Order patterns carefully**: Put specific patterns before general ones
3. **Use meaningful variable names**: When binding values, use descriptive names
4. **Keep patterns simple**: Complex patterns can be hard to read and maintain
5. **Consider helper functions**: For complex match arms, extract logic into separate functions

```ard
// Good: extract complex logic
fn handle_error_case(error: Str) {
  log_error(error)
  send_notification(error)
  update_metrics()
}

match result {
  ok(value) => process_success(value)
  err(error) => handle_error_case(error)
}
```