theme: Oyster

# Unsafe at any Swift

### _Or, how to win friends and corrupt memory_

^ Swift pointer APIs, weird little corner of the stdlib

---

## Maximum verbosity!

- `UnsafePointer<Pointee>`
- `UnsafeRawPointer`
- `UnsafeMutableRawPointer`
- `UnsafeMutableBufferPointer<Pointee>`
- `UnsafeRawCacaoArtisanalHandBuffedPoin...`

🤯

^ Swift is not afraid of verbosity, this kind of affects (infests?) this API.

^ Fortunately for us, it is fairly logical.

---

## Foggy Pointer Breakdown

They are all

# [fit] `Unsafe`

^ First thing to notice. Key point here, at the front of all the typenames

---

## Foggy Pointer Breakdown

- They are all "`Unsafe`"

## [fit] Swift is a "safe" language

^ Contrast to this well-publicized aspect of Swift

---

## [fit] Danger! Danger!
<br/>
What is "safety" here?

![left fit](danger-robot.jpg)


^ Let's talk about what safe/unsafe means here

---

## Danger! Danger!

![](danger-robot.jpg)

- What is "safety" here?

# **Memory safety**

^ In the normal course of Swift programming, you are not directly interacting with allocations, deallocations, who owns what pieces of memory. In contrast to C, C++. Similar to other high-level langs (Python, Java); the language hides it from you.

--- 

## Danger! Danger!

![](danger-robot.jpg)

- What is "safety" here?
- **Memory safety**
- Array subscripting off-by-one crashes; isn't this unsafe?

```swift
let list = [1, 2, 3]
print(list[3])    // 💥
```

^ Possibly the canonical example here. Try to read/write past the end of an array, and this ends up as a "fatal error" in Swift. It's a crash. That sucks. Shouldn't we handle this in a "safer" way?

---

## Danger! Danger!

![](danger-robot.jpg)

- What is "safety" here?
- **Memory safety**
- Array subscripting off-by-one crashes; isn't this unsafe?

# [fit] **NO**

^This is **safe** in the sense we're talking about here, because you are not: invoking Undefined Behavior; accessing memory that you do not own. Avoiding data corruption, user input buffer overflows (security issues), common sources of bugs

---

## Danger! Danger!

![](danger-robot.jpg)

- What is "safety" here?
- **Memory safety**
- Array subscripting off-by-one crashes; isn't this unsafe? **NO**
- Pointers are an interface to raw memory

^ Access to raw bytes. Via a Swift pointer you are avoiding ARC; if you have a pointer to an object you can arbitrarily mess with its innards.

---

## Foggy Pointer Breakdown

Pointers in Swift are all "`Unsafe`" ✅

Three other name components to consider

- `Mutable`
- `Raw`
- `Pointee`

^ Part of the reason for this API being so verbose; warning you off. Designers want this to be limited-use, scary.

^ Back to naming, three components. There's also "Buffer" that I'll get to later, these are core

---

## Mix 'n' match
<br/>
### [fit] `Mutable`

^ Next keyword, "mutable"

---

## Mix 'n' match

# `Mutable`

- Whether we can write _to the memory_

^ What is this?

---

## Mix 'n' match

# `Mutable`

- Whether we can write _to the memory_
- Has mutation in its interface, non-`Mutable` doesn't
- `initialize`, `assign`, `deinitialize`, `set` subscript

^ A non-mutable pointer is a read-only pointer

---

## Mix 'n' match

# `Mutable`

- Whether we can write _to the memory_
- Has mutation in its interface, non-`Mutable` doesn't
- `initialize`, `assign`, `deinitialize`, `set` subscript

```swift
let p: UnsafePointer<Int8>           // const int8_t * q;
let q: UnsafeMutablePointer<Int8>    // int8_t * p;
```

^ Compare to C, `const` applied to the pointed-to thing, not the pointer itself

---

## Mix 'n' match

## `Mutable`, whether we can write _to the memory_

Conversion is via initializer

```swift
let p: UnsafePointer<Int8> = ...
let q = UnsafeMutablePointer(mutating: p)
let r = UnsafePointer<Int8>(q)
```

^ Equivalent to stripping `const` in C

---

## Mix 'n' match

## `Mutable`, whether we can write _to the memory_

Mutating the pointer _itself_ is `var` vs. `let`

```swift
var p: UnsafeMutablePointer<Int8>    // int8_t * p;
let q: UnsafeMutablePointer<Int8>    // int8_t * const q;
```

^ Like any other variable in Swift

---

## Mix 'n' match
<br/>
### [fit] `Raw` vs. `<Pointee>`

^ These two components are counterparts, mutually exclusive

---

## Mix 'n' match

- Raw bytes vs. known type of contents

^ Raw, we don't know what type is stored. Typed, we do; it can be any Swift type, although of course if you take a pointer to a memory-managed object ARC does not know (dangling pointer).

---

## Mix 'n' match

- Raw bytes vs. known type of contents
- Typed pointers, access to `var pointee: Pointee`
- `Raw`, arithmetic and converting to a typed form

^ Access to the pointee == dereferencing. The type is known, Swift lets you use it as any other instance of that type: properties, methods.

---

## Mix 'n' match

- Raw bytes vs. known type of contents
- Typed pointers, access to `var pointee: Pointee`
- `Raw`, arithmetic and converting to a typed form

```swift
var p: UnsafeRawPointer    // const void * p;
var q: UnsafePointer<Int8>    // const int8_t * q;
```

^ There is a `storeBytes(of:)` method on a mutable raw pointer

---

## You say `nil`, I say `NULL`

Nullability of the pointer is optionality

```swift
var p: UnsafeMutablePointer<Int8>    // int8_t * __nonnull
var p: UnsafeMutablePointer<Int8>?    // int8_t * __nullable
```

^ As with var/let, Swift maps its "native" notation to pointer semantics

---

## You say `nil`, I say `NULL`

Nullability of the pointer is optionality

```swift
var p: UnsafeMutablePointer<Int8>    // int8_t * __nonnull
var p: UnsafeMutablePointer<Int8>?    // int8_t * __nullable
```

`nil` for a pointer is actually `0x0`: `NULL`

^ Clever

---

## Point me a picture

Swift|C
---|---
`UnsafeRawPointer` | `const void *`
`UnsafeMutableRawPointer` | `void *`
`UnsafePointer<Int8>` | `const int8_t *`
`UnsafeMutablePointer<Int8>` | `int8_t *`
`let p: UnsafeRawPointer` | `const void * const p`
`var p: UnsafeRawPointer` | `const void * p`
`UnsafePointer<Int8>?` | `const int8_t * __nullable`
`UnsafePointer<Int8>` | `const int8_t * __nonnull`

^ Summary of the terms and their corresponding C declaration

---

## What _is_ that thing?

- `Raw` pointer is just bytes

---

## What _is_ that thing?

- `Raw` pointer is just bytes
- Allocation is equivalent to `malloc`

```swift
let alignment = MemoryLayout<Int32>.alignment
let bytes = UnsafeMutableRawPointer.allocate(byteCount: 16,
                                             alignment: alignment)
```

^ Get space for 4 Int32s, aligned appropriately. Helpful `MemoryLayout` struct, compare to `__sizeof` `__alignof` from C

---

## What _is_ that thing?

To give the pointer a type, "bind" the memory

```swift
let bytes = UnsafeMutableRawPointer.allocate(byteCount: count,
                                             alignment: alignment)
let size = MemoryLayout<Int32>.size
let vals = bytes.bindMemory(to: Int32.self, capacity: count / size)
// vals is `UnsafeMutablePointer<Int32>`, pointing to the same memory
```

^ Swift has possibly a different "memory model" than C. Notion of memory being "bound" to a specific type. Dire warnings in docs about binding and undefined behavior.

^ One use case here is to have a memory pool from which you dispense chunks of typed memory. As with everything else, you have to track manually.

---

## What _is_ that thing?

Memory may already have a type, in which case it can be "assumed".

```swift
// Raw pointer from elsewhere, memory *known* to contain Int32s
let bytes = //...
let vals = bytes.assumingMemoryBound(to: Int32.self)
// vals is again `UnsafeMutablePointer<Int32>`,
// pointing to the same memory
```

^ The actual difference between "assume" and "bind" is not completely clear. My best understanding is that it's groundwork for stricter runtime behavior.

---

## Get on the buf(fer)!

`Unsafe(Mutable)BufferPointer`

^ One more kind of pointer. No direct C equivalent; this is a "nice thing" that Swift adds.

---

## Get on the buf(fer)!

# `Unsafe(Mutable)BufferPointer`
- Represents contiguous memory
- `Collection` interface

^ This is Swift's regular `Collection` protocol, everything available: iteration, `contains(where:)`, `prefix(_:)`

---

## Get on the buf(fer)!

# `Unsafe(Mutable)BufferPointer`
- Represents contiguous memory
- `Collection` interface
- Cannot create directly; must have already-allocated pointer

```swift
let count = 10
let base = UnsafeMutablePointer<Int8>.allocate(capacity: count)
let buf = UnsafeMutableBufferPointer(start: base, count: count)
```

^ As noted in the docs, the buffer does not own the memory, although strangely you can deallocate through it.

---

## Cleanup on aisle `ENOMEM`

- This is manual memory management!
- ARC does not apply

---

## Cleanup on aisle `ENOMEM`

- This is manual memory management!
- ARC does not apply
- Avoid leaking
- Don't forget to `deallocate` whatever you `allocate`

```swift
bytes.deallocate()
```

^ Call method on the pointer to return the memory. As with `allocate`/`malloc`, this is equivalent to `free` in C.

---

# It's not always rude to point

- C APIs, including Apple's for audio, video
- "System" programming
- Web server framework
- Language implementation

^ What do we use this for? C interop is going to be the main reason. Low-level, performance, controlling memory allocation (SwiftNIO)

---

## FIN

# [fit] 👉 ☝️ 👇 👈
