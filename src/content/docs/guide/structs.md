---
title: Structs
description: Learn about defining and using structs, methods, and static functions in Ard.
---

## Defining Structs

Structs group related data together:

```ard
struct Person {
  name: Str
  age: Int
  email: Str
}
```

## Creating Struct Instances

```ard
let person = Person {
  name: "Alice"
  age: 30
  email: "alice@example.com"
}
```

## Accessing Fields

Use dot notation to access struct fields:

```ard
let name = person.name        // "Alice"
let age = person.age          // 30
io::print("Hello, {person.name}!")
```

## Field Visibility

All struct fields are public by default and can be accessed from other modules:

```ard
// In user.ard
struct User {
  id: Int
  username: Str
  email: Str
}

// In main.ard
use myproject/user

let u = User { id: 1, username: "alice", email: "alice@example.com" }
io::print(u.username)  // Accessible from another module
```

## Methods

Use `impl` blocks to define methods on structs:

```ard
struct Rectangle {
  width: Float
  height: Float
}

impl Rectangle {
  fn area() Float {
    @width * @height
  }
  
  fn perimeter() Float {
    2.0 * (@width + @height)
  }
  
  fn is_square() Bool {
    @width == @height
  }
}
```

### The `@` Prefix

Within methods, use `@` prefix to access the struct's fields:

```ard
impl Person {
  fn get_intro() Str {
    "My name is {@name} and I am {@age.to_str()} years old"
  }
  
  fn is_adult() Bool {
    @age >= 18
  }
}
```

### Methods with Parameters

Methods can accept additional parameters:

```ard
impl Person {
  fn greet(other: Person) Str {
    "Hello, {other.name}! I'm {@name}."
  }
  
  fn celebrate_birthday() {
    @age =+ 1
  }
}
```

Note: The `celebrate_birthday` method would require the struct instance to be mutable.

## Using Methods

Call methods on struct instances:

```ard
let rect = Rectangle { width: 5.0, height: 3.0 }
let area = rect.area()           // 15.0
let perimeter = rect.perimeter() // 16.0
let is_square = rect.is_square() // false

let alice = Person { name: "Alice", age: 25, email: "alice@example.com" }
let intro = alice.get_intro()    // "My name is Alice and I am 25 years old"
```

## Static Functions

Define static functions using the `::` operator in the function name:

```ard
struct Todo {
  title: Str
  completed: Bool
}

// Static constructor function
fn Todo::new(title: Str) Todo {
  Todo { title: title, completed: false }
}

// Static utility function
fn Todo::completed_count(todos: [Todo]) Int {
  mut count = 0
  for todo in todos {
    if todo.completed {
      count =+ 1
    }
  }
  count
}
```

## Using Static Functions

Call static functions using the `::` operator:

```ard
let task = Todo::new("Learn Ard")  // Creates Todo { title: "Learn Ard", completed: false }

let todos = [
  Todo::new("Task 1"),
  Todo { title: "Task 2", completed: true }
]

let completed = Todo::completed_count(todos)  // 1
```

## Method Privacy

Methods can be made private using the `private` keyword:

```ard
impl User {
  fn get_display_name() Str {
    format_name(@username)  // Calls private method
  }
  
  private fn format_name(name: Str) Str {
    "User: {name}"
  }
}
```

Private methods can only be called from within the same module.

## Struct Composition

Structs can contain other structs:

```ard
struct Address {
  street: Str
  city: Str
  zip_code: Str
}

struct Employee {
  person: Person
  address: Address
  employee_id: Int
}

let emp = Employee {
  person: Person { name: "Bob", age: 35, email: "bob@company.com" }
  address: Address { street: "123 Main St", city: "Anytown", zip_code: "12345" }
  employee_id: 1001
}

// Accessing nested fields
let employee_name = emp.person.name
let employee_city = emp.address.city
```

## Mutable Structs

To modify struct fields, the struct instance must be mutable:

```ard
mut person = Person { name: "Charlie", age: 20, email: "charlie@example.com" }
person.age = 21           // OK, person is mutable
person.email = "new@example.com"  // OK

let immutable_person = Person { name: "Dave", age: 25, email: "dave@example.com" }
// immutable_person.age = 26  // Error: cannot modify immutable struct
```

## Pattern Matching with Structs

Structs can be used in pattern matching (future feature):

```ard
// Future syntax
match user {
  User { admin: true, .. } => "Admin user"
  User { name: "guest", .. } => "Guest user"
  _ => "Regular user"
}
```