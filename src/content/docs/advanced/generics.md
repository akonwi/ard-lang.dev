---
title: Generics
description: Learn about generic programming in Ard using type parameters.
---

## Overview

Generics allow writing code that works with multiple types while maintaining type safety. Ard uses a simple syntax for generic types that doesn't require explicit declarations.

## Generic Syntax

Generic types begin with `$` in function and struct declarations:

```ard
fn map(list: [$A], transform: fn($A) $B) [$B] {
  let result: [$B] = []
  for item in list {
    result.push(transform(item))
  }
  result
}
```

In this example, `$A` and `$B` are generic type parameters. The function accepts a list of type `$A` and returns a list of type `$B`.

## Type Inference

The compiler attempts to infer generic types from usage:

```ard
fn identity(value: $T) $T {
  value
}

let number = identity(42)        // $T inferred as Int
let text = identity("hello")     // $T inferred as Str
let flag = identity(true)        // $T inferred as Bool
```

## Explicit Type Arguments

When type inference isn't sufficient, provide explicit type arguments:

```ard
let ints = [1, 2, 3]
let floats = map<Int, Float>(ints, fn(x: Int) Float { x.to_float() })
```

Type arguments correspond to the order of generics in the function signature.

## Generic Structs

Structs can also be generic:

```ard
struct Container<$T> {
  value: $T
}

let int_container = Container { value: 42 }
let str_container = Container { value: "hello" }
```

## Generic Functions Examples

### List Operations

```ard
fn filter(list: [$T], predicate: fn($T) Bool) [$T] {
  let result: [$T] = []
  for item in list {
    if predicate(item) {
      result.push(item)
    }
  }
  result
}

fn fold(list: [$T], initial: $R, reducer: fn($R, $T) $R) $R {
  mut accumulator = initial
  for item in list {
    accumulator = reducer(accumulator, item)
  }
  accumulator
}

// Usage
let numbers = [1, 2, 3, 4, 5]
let evens = filter(numbers, fn(n: Int) Bool { n % 2 == 0 })
let sum = fold(numbers, 0, fn(acc: Int, n: Int) Int { acc + n })
```

### Optional Value Handling

```ard
fn map_maybe(maybe: $T?, transform: fn($T) $R) $R? {
  match maybe {
    value => maybe::some(transform(value))
    _ => maybe::none()
  }
}

fn flat_map_maybe(maybe: $T?, transform: fn($T) $R?) $R? {
  match maybe {
    value => transform(value)
    _ => maybe::none()
  }
}

// Usage
let maybe_number: Int? = maybe::some(42)
let maybe_doubled = map_maybe(maybe_number, fn(n: Int) Int { n * 2 })
```

### Result Transformations

```ard
fn map_result(result: $T!$E, transform: fn($T) $R) $R!$E {
  match result {
    ok(value) => Result::ok(transform(value))
    err(error) => Result::err(error)
  }
}

fn and_then(result: $T!$E, next: fn($T) $R!$E) $R!$E {
  match result {
    ok(value) => next(value)
    err(error) => Result::err(error)
  }
}

// Usage
fn parse_and_double(text: Str) Int!Str {
  let parsed = text.to_int().or_err("Invalid number")
  map_result(parsed, fn(n: Int) Int { n * 2 })
}
```

## Generic Structs with Methods

```ard
struct Pair<$A, $B> {
  first: $A
  second: $B
}

impl<$A, $B> Pair<$A, $B> {
  fn swap() Pair<$B, $A> {
    Pair { first: @second, second: @first }
  }
  
  fn map_first(transform: fn($A) $C) Pair<$C, $B> {
    Pair { first: transform(@first), second: @second }
  }
  
  fn map_second(transform: fn($B) $C) Pair<$A, $C> {
    Pair { first: @first, second: transform(@second) }
  }
}

// Usage
let pair = Pair { first: 42, second: "hello" }
let swapped = pair.swap()  // Pair<Str, Int>
let mapped = pair.map_first(fn(n: Int) Str { n.to_str() })  // Pair<Str, Str>
```

## Generic Collections

### Stack Implementation

```ard
struct Stack<$T> {
  items: [$T]
}

impl<$T> Stack<$T> {
  fn new() Stack<$T> {
    Stack { items: [] }
  }
  
  fn push(item: $T) {
    @items.push(item)
  }
  
  fn pop() $T? {
    match @items.len() {
      0 => maybe::none()
      _ => {
        let item = @items.last()
        @items.remove_last()
        maybe::some(item)
      }
    }
  }
  
  fn peek() $T? {
    match @items.len() {
      0 => maybe::none()
      _ => maybe::some(@items.last())
    }
  }
  
  fn is_empty() Bool {
    @items.len() == 0
  }
}

// Usage
let mut int_stack = Stack::new<Int>()
int_stack.push(1)
int_stack.push(2)
int_stack.push(3)

let top = int_stack.pop()  // Some(3)
```

## Type Constraints (Future Feature)

Generic types may be constrained with trait bounds:

```ard
// Future syntax
fn sort<$T: Comparable>(list: [$T]) [$T] {
  // Implementation using $T's comparison methods
}

fn debug_print<$T: String>(item: $T) {
  io::print(item.to_str())
}
```

## Type Inference Algorithm

The compiler uses the following approach for type inference:

1. **Collect generic types** from function signatures
2. **Apply explicit type arguments** if provided
3. **Infer from argument types** and usage context
4. **Refine generic types** based on constraints
5. **Report errors** if inference is incomplete

```ard
fn example(a: $T, b: $T) $T {
  // Both parameters must be the same type
  a
}

let result1 = example(1, 2)        // $T inferred as Int
let result2 = example("a", "b")    // $T inferred as Str
// let invalid = example(1, "a")   // Error: type mismatch
```

## Best Practices

### 1. Use Descriptive Generic Names

```ard
// Good: descriptive names
fn convert<$Input, $Output>(value: $Input, converter: fn($Input) $Output) $Output {
  converter(value)
}

// Less clear: single letters
fn convert<$A, $B>(value: $A, converter: fn($A) $B) $B {
  converter(value)
}
```

### 2. Minimize Generic Complexity

```ard
// Good: simple and focused
struct Result<$Value, $Error> {
  // ...
}

// Overly complex: too many generics
struct ComplexContainer<$A, $B, $C, $D, $E> {
  // Hard to understand and use
}
```

### 3. Provide Type Hints When Needed

```ard
// Help the compiler with explicit types
let numbers: [Int] = []
let results = map(numbers, fn(n: Int) Str { n.to_str() })

// Or use type arguments
let results = map<Int, Str>(numbers, fn(n) { n.to_str() })
```

### 4. Consider Default Types (Future Feature)

```ard
// Future syntax with defaults
struct Config<$Storage = FileStorage> {
  storage: $Storage
}

let config = Config {}  // Uses FileStorage by default
let custom_config = Config<DatabaseStorage> { storage: db_storage }
```