---
layout: post
title: Ownership & Borrowing
date: 2016-01-01 12:03:00
---

One of the most interesting features of Rust is its ownership system.  In Rust, variables are owned by their binding; so when variable binding goes out of scope, the data the variable is bound to is, deterministically, deallocated.  This is how Rust is able to function without a garbage collector or explicit freeing of memory.

<!--more-->

```rust
fn main() {
    let inta;
    let intb;
    let mut vector: Vec<&i32> = vec![];

    inta = 10;
    vector.push(&inta);

    intb = 10;
    vector.push(&intb);

    // Declarations are deallocated in reverse order
    // Vector is automatically deallocated, along with its heap
    // inta is deallocated
    // intb is deallocated
}
```

### Moving Ownership

Rust allows the ownership of a value to be transferred between bindings, but only one binding to a value is allowed.  This means moving a value to a new binding or lexical scope invalidates it for the current binding or scope.

For Example:
```rust
fn print(msg: String) {
    println!("{}", msg);

    // msg is deallocated here
}

fn main() {
    let message = String::from("Some Message");

    // Value moved into `print` function here
    print(message);

    // So it cannot be used here anymore
    println!("{}", message); // This fails
}
```

The rule specifying there may be only one binding to any resource helps ensure Rust is safe from data races.  If you can't have two mutable pointers to the same section of memory, you can't have data races!

That said, Rust is a systems language!  If I want to fiddle with some bits that have multiple mutable references, then I better dang well be able to.  Rust supports doing everything C does, using the `unsafe` code blocks.  The point is that, instead of making poor code easy to write, Rust makes poor code more difficult to write.  This means a majority of code is safe by default, and the unsafe parts can be abstracted away and more heavily vetted.

#### The Copy Trait

Sometimes moving ownership isn't the best option though.  For something like an `i32`, it would be just as quick to copy the value as it would be to move it.  Rust does exactly this by implementing the `Copy` trait for the `Int` struct.  Rust knows that because `i32` implements `Copy` Rust should use that instead of transferring ownership.  This is one of the many ways Rust's powerful type system is used.

### Borrowing

Rust wouldn't be a very useful language if you could only have one binding for any resource.  This is why Rust allows borrowing, the temporary transfer of ownership.

Here's an example:

```rust
use std::fmt::Display;

fn main() {
    let values = vec!["A", "B", "C"];

    print_values(&values);

    println!("First value: {}", values[0]);

    // values is deallocated here
}

fn print_values<T: Display>(values: &Vec<T>) {
    for val in values {
        println!("{}", val);
    }
}
```

In the above example, `values` is *borrowed* by `print_values`.  When the `print_values` function is finished it returns ownership back to the `main` function, instead of deallocating `values`.  Because a standard borrow is immutable, you can loan a value as many times as you want.

That said, if there are any immutable references to a resource, the resource cannot be modified; if it were possible to modify a resource with many references to it, access to the resource could potentially lead to a data race.

#### Mutable borrows

It is also possible to loan a mutable value of a resource:

```rust
fn main() {
    let mut values = vec![1, 2, 3];

    println!("Before:");
    for &val in values.iter() {
        println!("{}", val);
    }

    double_values(&mut values);

    println!("After:");
    for val in values {
        println!("{}", val);
    }

    // values is deallocated here
}

fn double_values(values: &mut Vec<i32>) {
    for val in values {
        *val *= 2;
    }
}
```

Unlike with immutable borrows, a resource may only be borrowed mutably once.

### Thinking about borrows

It can be useful to think of borrows by their scope.  The below snippet is completely valid, for example:

```rust
fn main() {
    let mut vector = vec!["a", "b"];

    {
        let v2 = &mut vector;
        v2.push("c");
    }

    println!("{}", vector.join(", "));
}
```

## Why?

It seems as though a lot of flexibility is given up with ownership, so why include it in Rust?  Ownership and borrowing completely prevent iterator invalidation and use-after free bugs.  Two entire classes of errors can be eliminated by following the relatively simple rules of Rust.  Of course if you're just writing a a little helper script or command line tool, this may not be a big deal, it may not even be worth the effort.  But if you're writing the foundational cryptographic primitives library for the internet, or critical-path code for life-and-death situations, being able to guarentee your program can't fail for these reasons allows more time to be spent auditing the actual logic of the code.
