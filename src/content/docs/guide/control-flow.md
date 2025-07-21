---
title: Control Flow
description: Learn about conditional statements, loops, and pattern matching in Ard.
---

## Conditional Statements

### If-Else

```ard
if temperature > 30 {
  io::print("It's hot!")
} else if temperature < 10 {
  io::print("It's cold!")
} else {
  io::print("Nice weather!")
}
```

Conditions must be boolean expressions. There are no truthy/falsy conversions.

### Conditional Expressions

If-else can be used as expressions:

```ard
let status = if logged_in { "Welcome" } else { "Please log in" }

let max = if a > b { a } else { b }
```

## Loops

### For Loops

#### Iterating Over Collections

```ard
let fruits = ["apple", "banana", "cherry"]
for fruit in fruits {
  io::print("I like {fruit}")
}
```

#### Iterating Over Maps

```ard
let scores: [Str:Int] = ["Alice": 95, "Bob": 87, "Carol": 92]
for name, score in scores {
  io::print("{name} scored {score.to_str()}")
}
```

#### Numeric Ranges

```ard
// Inclusive range
for i in 1..10 {
  io::print(i.to_str())  // Prints 1, 2, 3, ..., 10
}

// With step (if supported)
for i in 0..100 step 10 {
  io::print(i.to_str())  // Prints 0, 10, 20, ..., 100
}
```

#### C-Style For Loop

```ard
for mut i = 0; i <= 5; i =+ 1 {
  io::print("Count: {i.to_str()}")
}
```

### While Loops

```ard
mut count = 0
while count < 10 {
  io::print("Count is {count.to_str()}")
  count =+ 1
}
```

## Match Expressions

Match expressions provide powerful pattern matching capabilities:

### Basic Matching

```ard
let message = match status {
  "active" => "User is active"
  "inactive" => "User is inactive"
  "pending" => "User registration pending"
  _ => "Unknown status"
}
```

### Integer Matching

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

### Boolean Matching

```ard
let response = match is_valid {
  true => "Proceed"
  false => "Error: invalid input"
}
```

### Mixed Patterns

```ard
let category = match value {
  0 => "zero"
  1..10 => "small number"
  42 => "the answer"
  100..1000 => "big number"
  _ => "something else"
}
```

### Enum Matching

```ard
enum Status { active, inactive, pending }

let message = match user_status {
  Status::active => "Welcome back!"
  Status::inactive => "Please reactivate account"
  Status::pending => "Account under review"
}
```

### Range Matching

Integer ranges are inclusive and support various patterns:

```ard
let age_group = match age {
  0..12 => "child"
  13..19 => "teenager"
  20..64 => "adult"
  65..120 => "senior"
  _ => "invalid age"
}
```

When patterns overlap, the first match wins:

```ard
let special = match number {
  21 => "legal drinking age"  // This matches first
  20..25 => "young adult"     // This won't match for 21
  _ => "other"
}
```

## Pattern Matching Order

Patterns are evaluated in the order they appear. More specific patterns should come before general ones:

```ard
// Good: specific cases first
let category = match value {
  0 => "zero"
  1 => "one"
  2..10 => "small"
  11..100 => "medium"
  _ => "large"
}

// Problematic: general case first
let category = match value {
  0..100 => "small to medium"  // This catches everything 0-100
  42 => "the answer"           // This will never match
  _ => "large"
}
```

## Loop Control

While Ard doesn't have `break` or `continue` keywords, control flow can be managed through:

1. **Conditional logic within loops**
2. **Functions with early returns**
3. **Match expressions for complex conditions**

```ard
// Using conditional logic
for item in items {
  if should_skip(item) {
    // Continue to next iteration (no explicit continue needed)
  } else {
    process(item)
  }
}

// Using functions for early exit
fn process_items(items: [Item]) Bool {
  for item in items {
    match validate(item) {
      true => process(item)
      false => false  // Early return from function
    }
  }
  true
}
```