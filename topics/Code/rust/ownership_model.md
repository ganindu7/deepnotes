---
layout: default
title: Ownership model
nav_order: 1
permalink: /topics/Code/rust/ownership_model
parent: Rust
grand_parent: Development
---

## Ownership model

Status: Draft (work in progress) · Updated: {{ page.path | file_date | date_to_string }}

Rust is a memory-safe language without a garbage collector. Memory is managed through an ownership model that weaves memory management into the architecture of the language itself.

In brief, the ownership model states:

1. Every value has an owner.
2. There can be only one owner for a given value at a time.
3. A value is dropped when its owner goes out of scope.

Let's expand.

### Every value has only one owner

```rust
fn main() {
    let _some_string = String::from("Hello World");
}
```

The statement above creates a `String` object from the input `"Hello World"` and also assigns ownership of the created object to the identifier `_some_string`.

Now what happens if we do the following assignment?

```rust
    let _another_string = _some_string;
```

This is a transfer of ownership. The string object we created at the beginning is now owned by `_another_string`, and because there can be only one owner, `_some_string` no longer owns anything.

To test this, let's use print statements.

Example 1:

```rust
fn main() {
    let _some_string = String::from("Hello Ganindu");
    let _another_string = _some_string;
    println!("{}", _another_string);
}
```

Result:

```
Hello Ganindu
```

However, if the print statement is changed to

```rust
    println!("{}", _some_string);
```

the compiler refuses to build it:

![rustc error E0382: borrow of moved value `_some_string`, with the help text suggesting `.clone()`]({{ '/topics/Code/rust/ownership_model/move-error.png' | relative_url }}){: width="700"}

The error occurs because the value sticks with its current owner, `_another_string`, not the previous owner `_some_string`, after the ownership transfer on line 3.

This "move by default" ownership pattern works from a memory-safety perspective but can be somewhat tedious from a programming perspective. We will later go into how Rust addresses this without contradicting its memory guarantees.

### Ways to work with the ownership model

- **Borrowing** allows a value to be lent out instead of transferring ownership.
- **Cloning** lets you duplicate a value explicitly.
- **Copying** applies to simple plain data types that implement the `Copy` marker trait; assignment duplicates the value.

### Copying

```rust
fn main() {
    let some_int = 33;
    let _some_other_int = some_int;
    println!("{}", some_int);
}
```

Result:

```
33
```

This code compiles and generates the expected outcome. In this case the value is copied because of the `Copy` trait the data type possesses.

The `Copy` trait applies to data types with a fixed size known at compile time.

### Cloning

Cloning creates a replica in the heap, making the following possible.

```rust
fn main() {
    let some_string = String::from("This is just a string in the heap");
    let _some_other_string = some_string.clone();
    println!("{}", some_string);
}
```

Output:

```
This is just a string in the heap
```

### Borrowing

Not written yet.
