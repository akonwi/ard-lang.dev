---
title: Introduction
description: Learn about Ard's philosophy, design goals, and key features.
---

## What's Ard?

- **A** **R**-eally **D**-ope Language
- [ard is slang for "alright."](https://www.dictionary.com/e/slang/ard/)
- [Irish and Scottish Gaelic word meaning 'high, lofty', 'above the ground, elevated'](https://www.oxfordreference.com/display/10.1093/oi/authority.20110803095422661)
- Ardbeg is my favorite scotch and I was drinking it when I came up with this name

## Language Description

Ard is a modern, statically-typed programming language designed for clarity, safety, and ease. It combines features from JavaScript, Swift, and Go while introducing its own unique characteristics.

## Design Goals

- **Readability**: Ard code should be easy to read and understand.
- **Simple**: There should be one obvious way to do things.
- **Safety**: The compiler should catch errors at compile time.
- **Reliable**: Runtime is in Go, so it's fast and efficient.
  - [Future] Compiles to Go for portability.

## Philosophy

Ard follows Go's philosophy for readability from left to right, rather than the usual spiraling syntax found in C-based languages. This design choice makes code more intuitive to read and understand.

## Key Characteristics

### No Exceptions
Ard does not have exceptions. Instead, errors are represented as values using the built-in `Result<$Val, $Err>` type. This approach makes error handling explicit and predictable.

### Static Typing with Inference
Variables and functions are statically typed, but the compiler can infer types in most cases, reducing verbosity while maintaining safety.

### Pattern Matching
Powerful `match` expressions allow for expressive conditional logic and data destructuring.

### Memory Safety
Ard provides memory safety without the overhead of garbage collection by leveraging Go's runtime efficiency.

## What Makes Ard Different

- **Left-to-right readability**: `age =+ 1` instead of `age += 1`
- **No return keyword**: The last expression in a function is automatically returned
- **Explicit error handling**: No hidden exceptions, all errors are values
- **Simple module system**: Absolute imports from project root
- **Async without keywords**: No `async`/`await` infection, just `async::start()`