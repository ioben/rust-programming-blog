---
layout: post
title: A Crash-Course on Rust
date: 2016-01-01 12:01:00
---

If you squint your eyes, Rust can look pretty similar to other C-family languages.  These similarities can help you get going quickly with Rust

<!--more-->

Rust is a strongly and statically typed programming language.


## Statement Separator

Rust uses the `;` as a statement separator. This is similar to how Smalltalk used the `;` operator.


## Variable Bindings

Variable bindings can be created by using the `let` keyword.

```rust
fn main() {
  let pi = 3.14;
}
```

## Types

Rust has type inference, which means you don't have to explicitly tell the compiler the type of a variable. Of course, you are also free to do so.

```rust
fn main() {
  let num: f64 = 2.5;
}
```

Rust has many different numeric types:

 - i8 - Signed 8 bit integer
 - i16
 - i32
 - i64
 - u8 - Unsigned 8 bit integer
 - u16
 - u32
 - u64
 - isize - Pointer size of Signed Integer
 - usize - Pointer size of Unsigned Integer
 - f32 - 32 bit floating point
 - f64

Rust also has a boolean type 'bool',char type 'char', and String type str.


## Arrays

Rust supports arrays and are created using `[]`. Their size is computed at compile time. Arrays have a type signiture of `[objectType;size]`

```rust
	let people = ["Ann", "Jim", "Bob", "Katie"];
```

`people` would have a type signature of `[&str, 4]`.


## Slices

Slices are a reference of another data structure. This is very important in Rust because owning variables and borrowing are a big part of the language. A slice is a pointer to the beginning of the data and knows the length of the data, so that it can be referenced even if the object is not owned.

Slices are created with the `&` and `[]` operators. Say we have the array of people above, we can get a slice of those people

	let some_people = &people[2 .. 4];

Now, `some_people` is a slice of people and points to the elements "Bob" and "Katie".


## Functions

Functions in Rust are very similar to other languages. A simple adder function would look like this:

```rust
fn main(){
	fn add(a: i32, b:i32) -> i32 {
		a + b
	}
	let x = 2;
	let y = 4;
	println!("{} plus {} is {}", x, y, add(x,y));
}
```

The arguments of this function are `a` and `b`, both of type i32. The return value of a function is denoted by the -> operator, in this case, also an i32. It's important to note how the function returns a value, since we do not have a `return` keyword. Rust will always return the value of the last statement executed in the function, as long as it is not followed by a `;`. In which case, the function will return `()`. The following code will not compile, because it does not have a return value that matches its signature, `i32`

```rust
fn main() {
  fn add(a: i32, b: i32) -> i32 {
    a + b
  }

  let x = 2;
  let y = 4;
  println!("{} plus {} is {}", x, y, add(x, y));
}
```
