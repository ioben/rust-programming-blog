---
layout: post
title: What is Rust Anyway?
date: 2016-01-01 12:00:00
---

Well Rust is awesome!  More specifically Rust is a safe, concurrent, systems language which boasts an impressive collection of so called zero-cost abstractions.

<!--more-->

Rust guarantees memory safety, but that shouldn't be a guarantee you take on faith.  To achieve this Rust has three powerful concepts, Ownership, Borrowing, and Lifetimes.

Ownership is the concept of binding a variable to its scope.  When a variable is defined in a lexical scope, that variable is owned by its lexical scope.  When the scope is exited, the variable is automatically released, unless it was _moved_ to another lexical scope (which means it would no longer be available in the first scope).

It may be obvious to see why this would be jarring to newcomers of Rust.  For example the following code will __not__ work:

```rust
fn print(msg: String) {
    println!("{}", msg);
}

fn main() {
    let msg = String::from("test");
    print(msg);
    println!("{}", msg);
}
```

In fact the above code won't even compile!  This is because when the `print` function is called, the `msg` variable is _moved_ to the `print` function's lexical scope, from the `main`.  So when the `println!` macro is called it no longer has access to `msg`, and so compilation will fail.

At first this may seem silly, but what if instead of just printing a line, the `print` function calls to low level C code that deallocates the given `msg` variable?  Well now the `println!` macro in `main` is trying to use-after-free!

Rust obviously can't know what `print` will do before running it, so it looks at the type signature.  The type signature says `msg: String`, which means `print` wants ownership of a `msg` variable of type `String`.  If instead the type signature were `msg: &str`, Rust would know that `print` only wanted a read-only _borrow_ of the `msg` variable.

```rust
fn print(msg: &str) {
    println!("{}", msg);
}

fn main() {
    let msg = String::from("test");
    print(&msg);
    println!("{}", msg);
}
```

That compiles!  But it's important to note the `print` function is only given a read-only reference to the `msg` variable, `main` still owns the variable.  This means manipulating the variable is out, you'd need to make a copy of it in memory and manipulate that, if necessary.

Finally, Rust has smart pointers with lifetimes.  Lifetimes are arguably the most difficult part of Rust for newcomers, even though they're simple in concept.  A lifetime dictates how long a variable must live for, this allows Rust to reason about the lifetimes of variables at compile time and allow the compilation to fail if, for example, there is a potential use-after-free.

When new Rustaceans complain about fighting the compiler, more often than not they're talking about lifetime or ownership issues.  Error messages have improved considerably, but no error message can teach somebody how to think about and write their code in a new way, that can only happen with practice.

As you master the borrow system and lifetimes, Rust becomes a very powerful language, and lives up to its claims of being extremely safe.  This safety does come at a cost, there is a steep learning curve.

There is so much more to Rust though!  There are traits! And pattern matching! And type inference! And so much more, like easy binding to many other languages (like C, Ruby, and Python).  Rust is backed by Mozilla Labs, and they've began the herculean effort of writing a browser rendering engine in the language ([Servo](https://servo.org/)).  Other large companies are making good use of Rust such as Dropbox, Mozilla, and Canonical.
