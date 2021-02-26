---
title: "Iterate enum by reference in Rust"
date: 2021-02-12T06:38:50+01:00
draft: true
---

# Iterate enum by reference

## Code

```rust
#[derive(Debug)]
enum Color {
	White,
	Black,
}

fn main(){
	for color in &[Color::White, Color::Black] {
		if let Color::White = color {
			println!("{:?}", color);
		}
		
		if let Color::White = *color {
			println!("{:?}", color);
		}
	}
}
```

https://play.rust-lang.org/?version=stable&mode=debug&edition=2018&gist=c4a9b9cff781ba084809535a55f24171

## Plain code

```
#[derive(Debug)]
enum Color {
	White,
	Black,
}

fn main(){
	for color in &[Color::White, Color::Black] {
		if let Color::White = color {
			println!("{:?}", color);
		}
		
		if let Color::White = *color {
			println!("{:?}", color);
		}
	}
}
```