---
layout: post
title: Lifetimes
date: 2016-01-01 12:04:00
---

Closely related to both the borrow system and the type system, Rust has lifetimes which dictate how long a variable is accessible.

Every borrow in Rust has a lifetime which dictates the minimum lifespan of the borrow; though in many cases the Rust compiler will let you omit (called lifetime elision) the lifetime when it can be determined automatically.

<!--more-->

A lifetime is part of the borrow system, and is enforced using the type system.  Lifetimes are used to help prevent use-after-free bugs.

For example this code has a use-after-free bug, and shouldn't compile:

```rust
fn main() {
    let mut v = vec!["a", "b"];

    {
        let s = String::from("c"); // s goes into scope
        v.push(&s); // The lifetime of s must be as long, or longer than the lifetime of v

        // s goes out of scope
    }

    println!("{}", v.join(", "));

    // v goes out of scope
}
```

The above fails because a reference of `s` is stored in `v`, and then `s` goes out of scope while the reference to `s` does not.  This means `v` now contains a reference to junk memory, and without the ownership system we would never have known!

## Lifetimes in functions

Functions can of course take references, which means functions must also deal with lifetimes.  The basic syntax is:

```rust
fn msg<'a>(string: &'a str) {
    println!("{}", string);
}

fn main() {
    msg("A message");
}
```

In the function definition of `msg`, there is a generic lifetime `'a` defined in the brackets, and it is used as part of the type of the `string` argument.  This would be read as "A refernce to a str with the lifetime a."

## Lifetimes in structs

Structs too can contain references, and therefore must handle lifetimes.  The syntax is very similar to functions:

```rust
use std::fmt::{self, Display};

struct Message<'a> {
  string: &'a str
}

impl<'a> Message<'a> {
    fn new(string: &'a str) -> Self {
        Message {
            string: string,
        }
    }
}

impl<'a> Display for Message<'a> {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{}", self.string)
    }
}

fn main() {
    let blah = String::from("The Message");
    let message;
    message = Message::new(&blah);

    println!("{}", message);
}
```

Here are some examples of elided lifetimes:

```rust
fn print(s: &str); // elided
fn print<'a>(s: &'a str); // expanded

fn substr(s: &str, until: u32) -> &str; // elided
fn substr<'a>(s: &'a str, until: u32) -> &'a str; // expanded

fn get_str() -> &str; // ILLEGAL, no inputs

fn frob(s: &str, t: &str) -> &str; // ILLEGAL, two inputs
fn frob<'a, 'b>(s: &'a str, t: &'b str) -> &str; // Expanded: Output lifetime is ambiguous

fn get_mut(&mut self) -> &mut T; // elided
fn get_mut<'a>(&'a mut self) -> &'a mut T; // expanded

fn args<T: ToCStr>(&mut self, args: &[T]) -> &mut Command; // elided
fn args<'a, 'b, T: ToCStr>(&'a mut self, args: &'b [T]) -> &'a mut Command; // expanded

fn new(buf: &mut [u8]) -> BufWriter; // elided
fn new<'a>(buf: &'a mut [u8]) -> BufWriter<'a>; // expanded
```
