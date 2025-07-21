---
title: Functions
description: Learn about function definition, parameters, return values, and advanced features in Ard.
---

## Function Definition

Functions are defined using the `fn` keyword:

```ard
fn greet(name: Str) Str {
  "Hello, {name}!"
}
```

## Parameters and Return Types

Function parameters require type annotations. Return types are specified after the parameter list:

```ard
fn add(a: Int, b: Int) Int {
  a + b
}

fn print_message(msg: Str) {
  // No return type specified - function doesn't return a value
  io::print(msg)
}
```

## Return Values

There is no `return` keyword in Ard. The last expression in a function is automatically returned:

```ard
fn multiply(x: Int, y: Int) Int {
  x * y  // This value is returned
}

fn get_status(code: Int) Str {
  match code {
    200 => "OK"
    404 => "Not Found"
    500 => "Server Error"
    _ => "Unknown"
  }  // Match expression result is returned
}
```

## Named Arguments

Functions can be called with named arguments, allowing parameters to be specified in any order:

```ard
fn create_user(name: Str, age: Int, email: Str) User {
  User { name: name, age: age, email: email }
}

// Positional arguments (order matters)
create_user("Alice", 25, "alice@example.com")

// Named arguments (order doesn't matter)
create_user(age: 30, email: "bob@example.com", name: "Bob")
```

**Important**: When using named arguments, all arguments must be named. Mixing positional and named arguments is not supported.

```ard
// This is NOT allowed:
create_user("Charlie", age: 35, email: "charlie@example.com")
```

## First-Class Functions

Functions are first-class values and can be used as arguments:

```ard
fn map(list: [Int], transform: fn(Int) Int) [Int] {
  let mapped: [Int] = []
  for item in list {
    mapped.push(transform(item))
  }
  mapped
}

fn double(x: Int) Int {
  x * 2
}

// Pass function as argument
let numbers = [1, 2, 3, 4]
let doubled = map(numbers, double)
```

## Anonymous Functions

Functions can be defined inline without names:

```ard
let squared = map([1, 2, 3], fn(x: Int) Int { x * x })

// Multi-line anonymous functions
let processed = map(numbers, fn(n: Int) Int {
  let temp = n * 3
  temp + 1
})
```

## Function Signatures

When declaring function types, use the `fn` syntax:

```ard
let operation: fn(Int, Int) Int = add
let printer: fn(Str) = io::print
let generator: fn() Int = get_random_number
```

## Closures

Anonymous functions can capture variables from their surrounding scope:

```ard
fn make_counter(start: Int) fn() Int {
  mut count = start
  fn() Int {
    count =+ 1
    count
  }
}

let counter = make_counter(10)
let first = counter()   // 11
let second = counter()  // 12
```

## Higher-Order Function Examples

```ard
// Filter function
fn filter(list: [Int], predicate: fn(Int) Bool) [Int] {
  let result: [Int] = []
  for item in list {
    match predicate(item) {
      true => result.push(item)
      false => {}
    }
  }
  result
}

// Usage
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
let evens = filter(numbers, fn(n: Int) Bool { n % 2 == 0 })
```

## Function Composition

```ard
fn compose(f: fn(Int) Int, g: fn(Int) Int) fn(Int) Int {
  fn(x: Int) Int {
    f(g(x))
  }
}

let add_one = fn(x: Int) Int { x + 1 }
let times_two = fn(x: Int) Int { x * 2 }
let add_then_double = compose(times_two, add_one)

let result = add_then_double(5)  // (5 + 1) * 2 = 12
```