---
title: Modules
description: Learn about Ard's module system, imports, and code organization.
---

## Module Basics

Each Ard file is a module that can be either a complete program or used by other modules. Imports are declared at the top of files using the `use` keyword.

```ard
use ard/io
use my_project/utils as helpers

io::print("Hello from main module")
helpers::calculate(42)
```

## Import Syntax

The basic import syntax uses absolute paths from the project root:

```ard
use path/to/module
use path/to/module as alias
```

By default, the imported module is available by the last segment of the path. Use `as` to provide a custom name.

## Standard Library

The Ard standard library consists of modules under the `ard/*` path:

```ard
use ard/io          // Input/output functions
use ard/json        // JSON parsing and serialization
use ard/http        // HTTP client functionality
use ard/async       // Asynchronous programming
use ard/maybe       // Maybe type utilities
use ard/result      // Result type utilities
```

## Project Structure

Import paths are always absolute from the project root, determined by the presence of an `ard.toml` file:

```
my_calculator/
├── ard.toml          # Project configuration
├── main.ard          # Entry point
├── utils.ard         # Utility functions
└── math/
    └── operations.ard # Math operations
```

### Project Configuration

The `ard.toml` file defines the project:

```toml
name = "my_calculator"
```

If no `ard.toml` file is present, the project name defaults to the root directory name.

## Import Examples

With the above project structure:

```ard
// In main.ard
use my_calculator/utils
use my_calculator/math/operations

fn main() {
  let result = operations::add(5, 3)
  utils::log("Calculation complete")
}
```

```ard
// In utils.ard
use ard/io

fn log(message: Str) {
  io::print("[LOG] {message}")
}

fn format_number(num: Int) Str {
  "Number: {num.to_str()}"
}
```

```ard
// In math/operations.ard
fn add(a: Int, b: Int) Int {
  a + b
}

fn multiply(a: Int, b: Int) Int {
  a * b
}

fn divide(a: Int, b: Int) Int!Str {
  match b == 0 {
    true => Result::err("Division by zero")
    false => Result::ok(a / b)
  }
}
```

## Module Aliases

Use aliases to avoid naming conflicts or shorten long paths:

```ard
use my_calculator/math/operations as math
use my_calculator/string/operations as strings

fn main() {
  let sum = math::add(5, 3)
  let text = strings::concat("Hello", "World")
}
```

## Public and Private Declarations

### Public by Default

All declarations in a module are public and accessible from other modules by default:

```ard
// In utils.ard
fn helper_function() Str {  // Public
  "This can be called from other modules"
}

let global_config = "config_value"  // Public
```

### Private Declarations

Use the `private` keyword to make declarations module-local:

```ard
// In utils.ard
fn public_function() Str {
  private_helper()  // OK: same module
}

private fn private_helper() Str {  // Private
  "This cannot be called from other modules"
}

private let secret_key = "abc123"  // Private
```

## Struct Field Visibility

All struct fields are public by default:

```ard
// In user.ard
struct User {
  id: Int          // Public
  username: Str    // Public
  email: Str       // Public
}

// Methods can be private
impl User {
  fn get_display_name() Str {  // Public
    format_name(@username)     // Calls private method
  }
  
  private fn format_name(name: Str) Str {  // Private
    "User: {name}"
  }
}
```

## Circular Dependencies

Ard prevents circular dependencies between modules. The module system enforces a directed acyclic graph (DAG) structure:

```ard
// This would cause a circular dependency error:
// a.ard imports b.ard
// b.ard imports a.ard
```

To resolve circular dependencies, extract shared functionality into a separate module:

```ard
// Instead of a.ard ↔ b.ard, use:
// a.ard → shared.ard ← b.ard
```

## Module Organization Best Practices

### 1. Logical Grouping

Organize related functionality into modules:

```
my_web_app/
├── ard.toml
├── main.ard
├── models/
│   ├── user.ard
│   ├── post.ard
│   └── comment.ard
├── handlers/
│   ├── auth.ard
│   ├── api.ard
│   └── web.ard
└── utils/
    ├── validation.ard
    ├── crypto.ard
    └── database.ard
```

### 2. Clear Naming

Use descriptive module and function names:

```ard
use my_web_app/utils/validation as validate
use my_web_app/models/user as user_model
use my_web_app/handlers/auth as auth_handler
```

### 3. Minimal Interfaces

Keep module interfaces focused and minimal:

```ard
// Good: focused interface
use my_app/crypto

let hash = crypto::hash_password("secret")
let valid = crypto::verify_password("secret", hash)

// Less ideal: kitchen-sink module
use my_app/utils

let hash = utils::hash_password("secret")
let config = utils::load_config()
let log = utils::create_logger()
let db = utils::connect_database()
```

### 4. Standard Library Conventions

Follow standard library naming patterns:

```ard
use ard/io     // Short, descriptive names
use ard/json   // Clear purpose
use ard/http   // Standard abbreviations
```

## Re-exports

While not directly supported, you can create convenience modules that group related functionality:

```ard
// In my_app/prelude.ard - common imports
use ard/io
use ard/json
use my_app/models/user
use my_app/utils/validation

// Re-export common functions
fn print(msg: Str) {
  io::print(msg)
}

fn parse_json(text: Str) Json!Str {
  json::parse(text)
}
```

Then import the prelude:

```ard
use my_app/prelude

prelude::print("Hello")
let data = prelude::parse_json("{\"name\": \"Alice\"}")
```