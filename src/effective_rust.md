# Types

- Make invalid states inexpressible in your types

instead of:
```rust
pub struct DisplayProps {
    pub x: u32,
    pub y: u32,
    pub monochrome: bool,
    // `fg_color` must be (0, 0, 0) if `monochrome` is true.
    pub fg_color: RgbColor,
}
```

use:

```rust
pub enum Color {
    Monochrome,
    Foreground(RgbColor),
}

pub struct DisplayProps {
    pub x: u32,
    pub y: u32,
    pub color: Color,
}
```

- Use marker traits to distinguish behaviors that cannot be expressed in the trait function signatures
```rust
pub trait Sort {
    /// Rearrange contents into sorted order.
    fn sort(&mut self);
}

/// Marker trait to indicate that a [`Sort`] sorts stably.
pub trait StableSort: Sort {}
```

- trait in Rust can be used in two ways:
  - as a trait bound
  - as a trait object
- Prefer `Result` to `Option` if an error may communicate anything useful
- Prefer `Result` and `Option` transforms over explicit `match` expressions
```rust
// instead of 
pub fn find_user(username: &str) -> Result<UserId, std::io::Error> {
    let f = match std::fs::File::open("/etc/passwd") {
        Ok(f) => f,
        Err(e) => return Err(From::from(e)),
    };
    // ...
}
// use
pub fn find_user(username: &str) -> Result<UserId, std::io::Error> {
    let f = std::fs::File::open("/etc/passwd")?;
    // ...
}
```
- Implement `From` trait for conversions, use `Into` trait for trait bounds
- Implement the builder pattern for complex data structures(consider `derive_builder` crate)
- Consider using iterator transforms instead of explicit loops unless:
  - the loop body is large and/or multifunctional
  - the loop body involves error conditions that result in early termination of the surrounding function
  - performance is vital

# Traits

- You should implement `Eq` whenever your implement `PartialEq` if the type do not have any float-related peculiarities
- Implement `Drop` for any types that hold resources that must be released, such as:
  - access to operating system resources
  - access to synchronization resources
  - access to raw memory
