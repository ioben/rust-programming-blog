---
layout: post
title: Mutability
---

In Rust, variables are by default, not mutable, for example, the following code will not compile:

	let a = 1;
	a = 2;

In order to modify a variable, you must declare it mutable when you are creating the variable binding, using the `mut` keyword

	let mut a = 1;
	a = 2;

As stated in the borrowing section, a variable binding can be borrowed as a read-only borrow using `&` to reference the variable. However, if you wanted a reference that can change, you can create a mutable reference by preceding the binding with `&mut`
