---
title: Async Programming
description: Learn about asynchronous programming in Ard using fibers and the async module.
---

## Overview

Ard takes inspiration from Go and Rust for async execution. From Go, Ard uses goroutines as the underlying async implementation. From Rust, Ard adopts safety guardrails for managing shared memory between execution contexts.

## Fibers vs Threads

Ard uses **fibers** for separate execution contexts instead of OS threads. Currently, all fibers are goroutines under the hood, meaning they are "green" threads managed by the Go runtime. This design keeps the possibility of OS-level threads open without naming conflicts.

## The `ard/async` Module

Async functionality is provided through the `ard/async` standard library module:

```ard
use ard/async
use ard/io
```

## Basic Sleep

From the current (main) fiber, a program can sleep for a duration in milliseconds:

```ard
use ard/async
use ard/io

fn main() {
  io::print("hello...")
  async::sleep(1000)  // Sleep for 1 second
  io::print("world!")
}
```

This will print "hello...", wait 1 second, then print "world!".

## No `async`/`await` Keywords

Note how there's no `async`, `await`, or `go` keyword. Ard avoids JavaScript's problem of infectious `async` functions where async spreads throughout the codebase.

## Concurrent Execution

Use `async::start()` to run code concurrently by providing a callback function:

```ard
use ard/async
use ard/io

fn main() {
  io::print("1")
  
  async::start(fn() {
    io::print("2")
  })
  
  io::print("3")
}
```

The output of this program is not guaranteed to have the numbers in order (could be "1", "3", "2" or "1", "2", "3") because there's no control over when the async callback runs.

## Waiting for Fibers

The `async::start()` function returns a fiber handle that provides a `.wait()` method:

```ard
use ard/async
use ard/io

fn main() {
  io::print("1")
  
  let fiber = async::start(fn() {
    async::sleep(5000)  // Sleep for 5 seconds
    io::print("2")
  })
  
  fiber.wait()  // Wait for the fiber to complete
  io::print("3")
}
```

Now the output will always be:
```
1
2
3
```

With a 5-second delay between "1" and "2".

## Practical Examples

### Concurrent HTTP Requests

```ard
use ard/async
use ard/http
use ard/io

fn fetch_url(url: Str, name: Str) {
  match http::get(url) {
    ok(response) => io::print("{name}: {response.status.to_str()}")
    err(error) => io::print("{name}: Error - {error}")
  }
}

fn main() {
  let fiber1 = async::start(fn() {
    fetch_url("https://api.github.com", "GitHub")
  })
  
  let fiber2 = async::start(fn() {
    fetch_url("https://httpbin.org/status/200", "HTTPBin")
  })
  
  // Wait for both to complete
  fiber1.wait()
  fiber2.wait()
  
  io::print("All requests completed")
}
```

### Producer-Consumer Pattern

```ard
use ard/async
use ard/io

fn producer(items: [Str]) {
  for item in items {
    io::print("Producing: {item}")
    async::sleep(500)  // Simulate work
  }
  io::print("Producer finished")
}

fn consumer(count: Int) {
  for i in 1..count {
    async::sleep(800)  // Simulate processing time
    io::print("Consumed item {i.to_str()}")
  }
  io::print("Consumer finished")
}

fn main() {
  let items = ["apple", "banana", "cherry", "date"]
  
  let producer_fiber = async::start(fn() {
    producer(items)
  })
  
  let consumer_fiber = async::start(fn() {
    consumer(4)
  })
  
  // Wait for both to complete
  producer_fiber.wait()
  consumer_fiber.wait()
}
```

### Background Task with Periodic Execution

```ard
use ard/async
use ard/io

fn periodic_task(interval_ms: Int, max_iterations: Int) {
  for i in 1..max_iterations {
    io::print("Background task iteration {i.to_str()}")
    async::sleep(interval_ms)
  }
}

fn main() {
  io::print("Starting background task...")
  
  let background = async::start(fn() {
    periodic_task(2000, 5)  // Run 5 times with 2-second intervals
  })
  
  // Do other work
  for i in 1..3 {
    io::print("Main task work {i.to_str()}")
    async::sleep(3000)
  }
  
  // Wait for background task to complete
  background.wait()
  io::print("All tasks completed")
}
```

## Error Handling in Async Code

Fibers handle errors independently. Errors in one fiber don't automatically propagate to others:

```ard
use ard/async
use ard/io

fn risky_operation() Int!Str {
  // Simulate a failing operation
  Result::err("Something went wrong")
}

fn safe_async_task() {
  match risky_operation() {
    ok(value) => io::print("Success: {value.to_str()}")
    err(error) => io::print("Error in async task: {error}")
  }
}

fn main() {
  let fiber = async::start(fn() {
    safe_async_task()
  })
  
  fiber.wait()
  io::print("Main thread continues normally")
}
```

## Best Practices

### 1. Keep Fibers Focused

Create fibers for specific, well-defined tasks:

```ard
// Good: specific task
let fetch_fiber = async::start(fn() {
  fetch_user_data(user_id)
})

// Less ideal: multiple unrelated tasks
let mixed_fiber = async::start(fn() {
  fetch_user_data(user_id)
  update_cache()
  send_email()
  log_activity()
})
```

### 2. Handle Errors Appropriately

Always handle potential errors in async code:

```ard
fn async_task_with_error_handling() {
  let fiber = async::start(fn() {
    match risky_async_operation() {
      ok(result) => process_result(result)
      err(error) => {
        log_error(error)
        // Handle gracefully
      }
    }
  })
  
  fiber.wait()
}
```

### 3. Use `.wait()` Judiciously

Don't wait immediately if you can do other work:

```ard
// Good: start all tasks, then wait
let fiber1 = async::start(fn() { task1() })
let fiber2 = async::start(fn() { task2() })
let fiber3 = async::start(fn() { task3() })

// Do other work here

fiber1.wait()
fiber2.wait()
fiber3.wait()

// Less efficient: wait immediately
let fiber1 = async::start(fn() { task1() })
fiber1.wait()  // Blocks immediately
let fiber2 = async::start(fn() { task2() })
fiber2.wait()  // Sequential execution
```

### 4. Avoid Shared Mutable State

Since fibers run concurrently, avoid sharing mutable state without proper synchronization:

```ard
// Problematic: shared mutable state
mut global_counter = 0

let fiber1 = async::start(fn() {
  global_counter =+ 1  // Race condition
})

let fiber2 = async::start(fn() {
  global_counter =+ 1  // Race condition
})
```

Instead, use message passing or return results from fibers to coordinate state changes.