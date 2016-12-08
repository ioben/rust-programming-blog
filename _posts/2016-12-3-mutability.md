---
layout: post
title: Mutability
date: 2016-01-01 12:05:00
---

In Rust, variables are by default, not mutable, for example, the following code will not compile:

```rust
	let a = 1;
	a = 2;
```

In order to modify a variable, you must declare it mutable when you are creating the variable binding, using the `mut` keyword

```rust
	let mut a = 1;
	a = 2;
```

As stated in the borrowing section, a variable binding can be borrowed as a read-only borrow using `&` to reference the variable. However, if you wanted a reference that can change, you can create a mutable reference by preceding the binding with `&mut`

## Interior vs Exterior Mutability

When something is called immutable in Rust it doesn't actually mean it can never be changed.

Consider `Arc<T>`, an atomic reference counted container, in the following example:

```rust
use std::sync::Arc;
use std::time;
use std::thread;

fn main() {
    let value = Arc::new("Hello!");

    for _ in 1..10 {
        let thread_value = value.clone();

        thread::spawn(move || {
            println!("{}", thread_value);
        });
    }

    thread::sleep(time::Duration::from_millis(100));
}
```

It's important to understand that Rust is first and foremost a memory safe language, and this helps inform all of the language's design decisions.  In Rust, something is immutable if it is safe to have two pointers to it.

So then, what is __interior mutability__?  Well, lets look at a `RefCell`:

```rust
use std::cell::RefCell;

fn main() {
  let x = RefCell::new(42);

  println!("{}", *x.borrow_mut());
}
```

A `RefCell` will hand out mutable references to the data contained within.  This is dangerous though, what if we called `borrow_mut` twice?  Well `RefCell` would panic; essentially `RefCell` implements the borrow-checker at runtime instead of compile-time.  The trade off is `RefCell` has a small amount of overhead at runtime.

## Field-level Mutability

Simply put, there is no such thing as field-level mutability in Rust.  Mutability is a property of a binding or reference in Rust.  This is one of the benefits of having structs and their implementations separated, it's easy to reason about both the mutable and immutable variants of a struct when the data and its operations are separated.
