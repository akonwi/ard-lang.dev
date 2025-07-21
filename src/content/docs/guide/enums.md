---
title: Enums
description: Learn about defining and using enums for representing discrete sets of values.
---

## Defining Enums

Enums enumerate a discrete set of named values. In Ard, enums are labeled integers and cannot have associated values:

```ard
enum Status {
  active,
  inactive,
  pending
}

enum HttpStatus {
  ok,
  not_found,
  server_error,
  bad_request
}
```

## Accessing Enum Values

Use the static operator `::` to access enum variants:

```ard
let current_status = Status::active
let response_code = HttpStatus::ok
```

The static operator avoids naming conflicts between enum variants and other identifiers in scope.

## Using Enums in Functions

```ard
fn get_status_message(status: Status) Str {
  match status {
    Status::active => "System is active"
    Status::inactive => "System is inactive"
    Status::pending => "System status pending"
  }
}

let message = get_status_message(Status::active)
```

## Pattern Matching with Enums

Enums work seamlessly with match expressions:

```ard
enum Priority {
  low,
  medium,
  high,
  critical
}

fn handle_task(priority: Priority) {
  match priority {
    Priority::low => io::print("Task can wait")
    Priority::medium => io::print("Task should be done today")
    Priority::high => io::print("Task needs attention soon")
    Priority::critical => io::print("Drop everything and handle this!")
  }
}
```

## Enum Comparison

Enums can be compared for equality:

```ard
let status1 = Status::active
let status2 = Status::active
let status3 = Status::pending

let same = status1 == status2      // true
let different = status1 == status3 // false
```

## Practical Examples

### State Machines

```ard
enum ConnectionState {
  disconnected,
  connecting,
  connected,
  error
}

fn handle_connection(state: ConnectionState) ConnectionState {
  match state {
    ConnectionState::disconnected => {
      io::print("Attempting to connect...")
      ConnectionState::connecting
    }
    ConnectionState::connecting => {
      // Simulate connection logic
      match connection_successful() {
        true => ConnectionState::connected
        false => ConnectionState::error
      }
    }
    ConnectionState::connected => {
      io::print("Already connected")
      ConnectionState::connected
    }
    ConnectionState::error => {
      io::print("Connection failed, retrying...")
      ConnectionState::disconnected
    }
  }
}
```

### HTTP Response Handling

```ard
enum HttpMethod {
  get,
  post,
  put,
  delete,
  patch
}

fn format_request(method: HttpMethod, path: Str) Str {
  let method_name = match method {
    HttpMethod::get => "GET"
    HttpMethod::post => "POST"
    HttpMethod::put => "PUT"
    HttpMethod::delete => "DELETE"
    HttpMethod::patch => "PATCH"
  }
  
  "{method_name} {path} HTTP/1.1"
}
```

### Configuration Options

```ard
enum LogLevel {
  debug,
  info,
  warn,
  error
}

enum Environment {
  development,
  staging,
  production
}

struct Config {
  log_level: LogLevel
  environment: Environment
  debug_mode: Bool
}

fn create_config(env: Environment) Config {
  let log_level = match env {
    Environment::development => LogLevel::debug
    Environment::staging => LogLevel::info
    Environment::production => LogLevel::warn
  }
  
  let debug_mode = match env {
    Environment::development => true
    Environment::staging => false
    Environment::production => false
  }
  
  Config {
    log_level: log_level,
    environment: env,
    debug_mode: debug_mode
  }
}
```

## Enum Limitations

Unlike some languages, Ard enums:
- Cannot have associated data/values
- Are essentially named integers
- Cannot have methods defined on them directly

For more complex data structures with variants, consider using type unions:

```ard
// Instead of enum with data, use type unions
type Result = Success | Error
type Success = { value: Str }
type Error = { message: Str }
```

## Best Practices

1. **Use descriptive names** for both the enum and its variants
2. **Group related states** together in a single enum
3. **Always handle all cases** in match expressions
4. **Use enums for finite sets** of known values
5. **Prefer type unions** when variants need associated data