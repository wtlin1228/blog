---
title: Baby Rustacean - Ownership
excerpt: Learn the Rust programming language
date: 2022-07-07
tags: [rust, ownership]
slug: baby-rustacean-ownership
featured: images/cover.jpg
---

![cover](./images/cover.jpg)

# Ownership

Why ownership?

- Rust automatically calls the `drop` function and cleans up the heap memory when the variable goes out of scope.
- It could cause problem if Rust frees the same memory twice.
- It's very expensive to copy the data on the heap if the data were large.

Ownership rules:

- Each value in Rust has an owner.
- There can only be one owner at a time.
- When the owner goes out of scope, the value will be dropped.

## Copy

Rust copies the stack-only data like `integers` and `boolean`.

```rust
let x = 5;
let y = x;

println!("x = {}, y = {}", x, y);
```

## Move

Rust moves the data on the heap like `String`, `Vec`.

```rust
let s1 = String::from("hello");
//  -- move occurs because `s1` has type `String`, which does not implement the `Copy` trait
let s2 = s1;
//       -- value moved here

println!("{s1}, {s2}");
//         ^^ value borrowed here after move
```

## Immutable Reference

Rust calls the action of creating a reference borrowing.

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize { // s is a reference to a String
    s.len()
} // Here, s goes out of scope. But because it does not have ownership of what
  // it refers to, it is not dropped.
```

## Mutable References

To allow a borrowed value to be modified, use mutable reference `&mut`.

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

## Reference Restrictions

1. If you have a mutable reference to a value, you can have no other references to that value.

```rust
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s;

println!("{}, {}", r1, r2);
```

2. If you have an immutable reference to a value, you can have no mutable reference to that value until the immutable reference is no longer used.

```rust
let mut s = String::from("hello");

let r1 = &s; // no problem
let r2 = &s; // no problem
let r3 = &mut s; // BIG PROBLEM

println!("{}, {}, and {}", r1, r2, r3);
```

```rust
let mut s = String::from("hello");

let r1 = &s; // no problem
let r2 = &s; // no problem
println!("{} and {}", r1, r2);
// variables r1 and r2 will not be used after this point

let r3 = &mut s; // no problem
println!("{}", r3);
```

## Dangling References

Definition: Freeing some memory while preserving a pointer to that memory.

```rust
fn main() {
    let reference_to_nothing = dangle();
    let reference_to_something = no_dangle();
}

// this function's return type contains a borrowed value, but there is no value for it to be borrowed from
fn dangle() -> &String {
    let s = String::from("hello");

    &s
}

fn no_dangle() -> String {
    let s = String::from("hello");

    s
}
```

## The Slice Type

Slices let you reference a contiguous sequence of elements in a collection rather than the whole collection. A slice is a kind of reference, so it does not have ownership.

```rust
fn main() {
    let s = String::from("hello");
    let slice = &s[0..2];
    let slice = &s[..2];

    let a = [1, 2, 3, 4, 5];
    let slice = &a[1..3];

    let mut a = [1, 2, 3, 4, 5];
    let mutable_slice = &mut a[1..3];
    mutable_slice[1] = 99;
    println!("{:?}", a) // [1, 2, 99, 4, 5];
}
```
