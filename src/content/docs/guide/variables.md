---
title: Variables & Constants
description: Learn about variable declaration, mutability, and type inference in Ard.
---

## Declaration Keywords

Ard uses two keywords for variable declaration:

- `let` for constants (immutable values)
- `mut` for variables (mutable values)

```ard
let name: Str = "Alice"    // Immutable
mut age = 30               // Mutable
```

## Type Inference

Variable types can be inferred from their initial values:

```ard
let count = 42           // Inferred as Int
let pi = 3.14           // Inferred as Float
let greeting = "Hello"   // Inferred as Str
let active = true       // Inferred as Bool
```

## Explicit Type Annotations

Types can be explicitly declared when needed:

```ard
let name: Str = "Bob"
let temperature: Float = 98.6
let items: [Int] = [1, 2, 3]
let mapping: [Str:Int] = ["a": 1, "b": 2]
```

## Immutability with `let`

Variables declared with `let` cannot be reassigned or mutated:

```ard
let x = 10
x = 20        // Error: cannot reassign immutable variable

let numbers = [1, 2, 3]
numbers.push(4)  // Error: cannot mutate immutable variable
```

## Mutability with `mut`

Variables declared with `mut` can be modified:

```ard
mut counter = 0
counter = 5           // OK
counter =+ 1          // OK, increment by 1
```

## Increment and Decrement

Ard uses a unique syntax for compound assignment operators, placing the `=` first for left-to-right readability:

```ard
mut value = 10

value =+ 5    // Equivalent to value = value + 5
value =- 2    // Equivalent to value = value - 2
value =* 3    // Equivalent to value = value * 3
value =/ 2    // Equivalent to value = value / 2
```

There are no `++` or `--` operators in Ard.

## Scoping Rules

Variables follow standard block scoping rules:

```ard
let outer = "global"

if some_condition {
  let inner = "local"
  // Both 'outer' and 'inner' are accessible here
}

// Only 'outer' is accessible here
// 'inner' is out of scope
```

## Shadowing

Variable shadowing is allowed, creating a new variable with the same name:

```ard
let x = 5
let x = x + 1    // Creates new variable, x is now 6
let x = "hello"  // Creates new variable with different type
```

## Constants vs Variables Guidelines

Use `let` by default and only use `mut` when the value needs to change:

```ard
let user_name = get_user_input()     // Won't change
mut attempt_count = 0                // Will increment
let max_attempts = 3                 // Configuration, won't change

for i in 1..max_attempts {
  attempt_count =+ 1
  // ...
}
```

This approach makes code more predictable and helps prevent accidental mutations.