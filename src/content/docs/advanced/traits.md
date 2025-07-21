---
title: Traits
description: Learn about defining and implementing traits for shared behavior in Ard.
---

## What are Traits?

Traits define shared behavior that can be implemented by different types. They are similar to interfaces in other languages but with some key differences. The Rust definition applies well to Ard:

> A trait defines the functionality a particular type has and can share with other types. We can use traits to define shared behavior in an abstract way.

## Defining Traits

Traits consist of method signatures that implementing types must provide:

```ard
trait String {
  fn to_str() Str
}
```

A trait can have multiple methods:

```ard
trait Drawable {
  fn draw()
  fn get_bounds() Rectangle
  fn is_visible() Bool
}
```

## Implementing Traits

Use `impl TraitName for TypeName` to implement a trait for a specific type:

```ard
struct Person {
  name: Str
  age: Int
}

impl String for Person {
  fn to_str() Str {
    "{@name} is {@age.to_str()} years old"
  }
}
```

Within trait implementations, use `@` to access the implementing type's fields.

## Using Traits

### As Function Parameters

Traits can be used as function parameter types to accept any type that implements the trait:

```ard
fn debug(thing: String) {
  io::print(thing.to_str())
}

let person = Person { name: "Alice", age: 30 }
debug(person)  // Calls Person's to_str() implementation
```

### With Different Types

Multiple types can implement the same trait:

```ard
struct Product {
  name: Str
  price: Float
}

impl String for Product {
  fn to_str() Str {
    "{@name}: ${@price.to_str()}"
  }
}

struct Order {
  id: Int
  items: [Product]
}

impl String for Order {
  fn to_str() Str {
    "Order #{@id.to_str()} with {@items.len().to_str()} items"
  }
}

// Both types can be used with functions expecting String trait
let product = Product { name: "Laptop", price: 999.99 }
let order = Order { id: 123, items: [product] }

debug(product)  // "Laptop: $999.99"
debug(order)    // "Order #123 with 1 items"
```

## Built-in Traits

Ard provides several built-in traits that types can implement:

### String Trait

The `String` trait enables string conversion and interpolation:

```ard
trait String {
  fn to_str() Str
}

// Built-in implementations exist for:
// Int, Float, Bool

let num = 42
let text = num.to_str()  // "42"
```

### Comparable Trait (Future)

```ard
trait Comparable {
  fn compare(other: Self) Int  // Returns -1, 0, or 1
  fn equals(other: Self) Bool
}
```

## Complex Trait Examples

### Display Formatting

```ard
trait Display {
  fn display() Str
  fn display_detailed() Str
}

struct User {
  id: Int
  username: Str
  email: Str
  active: Bool
}

impl Display for User {
  fn display() Str {
    "@{@username}"
  }
  
  fn display_detailed() Str {
    "User {@id.to_str()}: {@username} ({@email}) - {status()}"
  }
  
  fn status() Str {
    match @active {
      true => "Active"
      false => "Inactive"
    }
  }
}
```

### Serialization Trait

```ard
trait Serializable {
  fn serialize() Str
  fn deserialize(data: Str) Self!Str
}

struct Config {
  host: Str
  port: Int
  debug: Bool
}

impl Serializable for Config {
  fn serialize() Str {
    "host={@host},port={@port.to_str()},debug={@debug.to_str()}"
  }
  
  fn deserialize(data: Str) Config!Str {
    // Parse the serialized string and return Config or error
    let parts = data.split(",")
    // Implementation details...
    match parse_config_parts(parts) {
      ok(config) => Result::ok(config)
      err(error) => Result::err("Failed to deserialize: {error}")
    }
  }
}
```

### Collection Trait

```ard
trait Collection {
  fn size() Int
  fn is_empty() Bool
  fn contains(item: Self::Item) Bool
}

impl Collection for [Int] {
  fn size() Int {
    @.len()
  }
  
  fn is_empty() Bool {
    @.len() == 0
  }
  
  fn contains(item: Int) Bool {
    for element in @ {
      if element == item {
        return true
      }
    }
    false
  }
}
```

## Trait Bounds in Functions

Functions can require multiple traits:

```ard
// Future syntax for multiple trait bounds
fn process<T: Display + Serializable>(item: T) {
  io::print("Processing: {item.display()}")
  let serialized = item.serialize()
  save_to_file(serialized)
}
```

## Default Implementations (Future Feature)

Traits may provide default implementations:

```ard
trait Logger {
  fn log(message: Str)
  
  fn log_error(message: Str) {
    log("ERROR: {message}")  // Default implementation
  }
  
  fn log_info(message: Str) {
    log("INFO: {message}")   // Default implementation
  }
}

struct FileLogger {
  path: Str
}

impl Logger for FileLogger {
  fn log(message: Str) {
    // Only need to implement the required method
    write_to_file(@path, message)
  }
  
  // log_error and log_info are automatically available
}
```

## Trait Objects and Dynamic Dispatch (Future)

Store different types that implement the same trait:

```ard
// Future syntax
let drawables: [dyn Drawable] = [
  Circle { radius: 5.0 },
  Rectangle { width: 10.0, height: 20.0 },
  Triangle { base: 8.0, height: 6.0 }
]

for drawable in drawables {
  drawable.draw()  // Dynamic dispatch
}
```

## Best Practices

### 1. Keep Traits Focused

Design traits around a single responsibility:

```ard
// Good: focused trait
trait Renderable {
  fn render() Str
}

// Less ideal: too many responsibilities
trait SuperTrait {
  fn render() Str
  fn save_to_file() Bool
  fn send_over_network() Result<(), NetworkError>
  fn validate_data() Bool
}
```

### 2. Use Descriptive Names

Choose names that clearly indicate the trait's purpose:

```ard
trait Debuggable {     // Clear purpose
  fn debug_info() Str
}

trait Convertible {    // Clear capability
  fn convert_to(format: Str) Str!ConversionError
}
```

### 3. Consider Default Implementations

Provide sensible defaults when possible (future feature):

```ard
trait Cacheable {
  fn cache_key() Str
  
  fn cache_ttl() Int {
    3600  // Default: 1 hour
  }
  
  fn is_cacheable() Bool {
    true  // Default: cacheable
  }
}
```

### 4. Document Trait Requirements

Make trait contracts clear:

```ard
// Document expectations and invariants
trait Hashable {
  // Must return consistent hash for equal objects
  // Hash should remain stable across program runs
  fn hash() Int
  
  // Must be consistent with hash()
  // If a.equals(b), then a.hash() == b.hash()
  fn equals(other: Self) Bool
}
```