---
layout: post
title: Structs, Enums, Traits, and Implementations
date: 2016-01-01 12:02:00
---

Rust isn't functional or imperative; it's a little of both!  Because of this Rust's structure is a little different than anything you may have seen before.

<!--more-->

Structs
-------

Rust has three types of structs:

 - A unit struct

    ```
    struct Nil;
    ```

 - A tuple struct

   ```
   struct Pair(i32, i32);
   ```

 - And a regular struct

   ```
   struct Rectangle {
     x: f32,
     y: f32,
     width: f32,
     height: f32,
   }
   ```


Enums
-----

An Enum type in Rust can be one of any of the predefined possible values:

```rust
enum Color {
  RGB(u8, u8, u8),
  HSL(u8, u8, u8),
  CMYK(u8, u8, u8, u8),
}
```

Match and let statements allow pattern matching, and can be very useful when dealing with enums:

```rust
#[allow(dead_code)]
enum Color {
  RGB(u8, u8, u8),
  HSL(u8, u8, u8),
  CMYK(u8, u8, u8, u8),
}

fn main() {
  let white = Color::RGB(255, 255, 255);
  match white {
    Color::RGB(r, g, b) => println!("R: {}, G: {}, B: {}", r, g, b),
    Color::HSL(h, s, l) => println!("H: {}, S: {}, L: {}", h, s, l),
    Color::CMYK(c, m, y, k) => println!("C: {}, M: {}, Y: {}, K: {}", c, m, y, k),
  };

  if let Color::RGB(r, g, b) = white {
    println!("R: {}, G: {}, B: {}", r, g, b);
  }
}
```

Implementations
---------------

It's possible to implement functions for a `struct` or `enum`, this is done with the `impl` keyword:

```rust
struct Point(f32, f32);

#[allow(dead_code)]
impl Point {
  fn new(x: f32, y: f32) -> Self {
    Point(x, y)
  }
  fn newi(x: i32, y: i32) -> Self {
    Point(x as f32, y as f32)
  }

  fn dist(&self, point: Point) -> f32 {
    ((self.0 - point.0).powi(2) + (self.1 - point.1).powi(2)).sqrt()
  }
}

fn main() {
  let dist = Point::newi(10, 10).dist(Point::newi(15, 15));
  println!("Distance between (10, 10) and (15, 15) = {}", dist);
}

```

Traits
------

Traits can, in simple terms, be thought of as an interface for a `struct` or `enum`.  Instead of inheritance, which OOP languages have, in Rust interfaces are composed together.  A struct or enum can implement any number of traits, and functions require one or more traits be implemented for a specific argument.

```rust
trait Printable {
  fn print(&self) -> &str;
}

struct Message {
  string: String
}

impl Printable for Message {
  fn print(&self) -> &str {
    &self.string
  }
}

fn print<T: Printable>(printable: T) {
  println!("{}", printable.print());
}

fn main() {
  let message = Message { string: String::from("A Message") };
  print(message);
}
```

That is a bit verbose though, and we don't actually need to define a `Printable` trait, Rust already has one we can use:

```rust
use std::fmt::{self, Display};

struct Message(String);

impl Display for Message {
  fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
    write!(f, "{}", self.0)
  }
}

fn main() {
  let message = Message(String::from("A Message"));

  println!("{}", message);
}
```
