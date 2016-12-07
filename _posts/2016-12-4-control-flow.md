---
layout: post
title: Control Flow
---

Rust's control flow is similar to that of other languages.

### If Statement
<!-- more -->
```rust
fn main() {
	let x = 2;
	let y = 4;
	if x < y {
		println!("{} is less than {}",x , y);
	}
	else if x > y {
		println!("{} is greater than {}", x, y);
	}
	else {
		println!("{} is the same as {}", x, y);
	}
}
```

The main difference you may see between Rust and other programming languages is that Rust does not use parenthesis.

### Loops

Rust, by default, has an infinite loop, using the keyword, `loop`
```rust
fn main(){
	let mut x = 1;
	loop{
		x = x+1;
	}
}
```

Rust makes use of the `break` and `continue` keywords, so that the above `loop` keyword can be useful.

#### For Loop

The for loop looks a bit different then other languages, but is a bit more simple.

```rust
fn main() {
	let x = 1;
	for y in 0..20{
		if y % 2 == x{
			println!("{} is odd!", y);
		} else {
			println!("{} is even!", y);
		}
	}
}
```
#### While Loop

In Rust, the while loop is similar to what we've seen in other languages, however, like the if statement and for loop, parenthesis are not used.

```rust
fn main() {
	let x = 1;
	let mut y = 0;
	while y<=20 {
		if y % 2 == x{
			println!("{} is odd!", y);
		} else {
			println!("{} is even!", y);
		}
	y += 1;
	}
}
```

