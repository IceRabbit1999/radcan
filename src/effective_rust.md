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
- `trait Shape: Draw` is better expressed as `Shape` also-implements `Draw`, instead of `Shape` is-a `Draw`
- Trait object safety:
  - trait's methods must NOT be generic
  - trait's methods must not involve type that includes `Self`, except for the receiver
- A degree of tension in the trait design:
  - To the implementors: it's better for a trait to have the absolute minimum number of methods to achieve its purpose
  - To user: it's helpful to provide a range of variant methods that cover all of the common ways that the trait might be used
  - This tension can be balanced by including the wider range of methods, but with default implementations provided

# Concepts

- When there are two lifetimes `'a` that are the 'same', that just means that the output lifetime has to be contained within the life times of both of the inputs
```rust
pub fn smaller<'a>(left: &'a Item, right: &'a Item) -> &'a Item {
    // ...
}
```
Put it another way, the output lifetime has to be subsumed within the smaller of the lifetimes of the two inputs
```rust
{
    let outer = Item { contents: 7 };
    {
        let inner = Item { contents: 8 };
        {
            let min = smaller(&inner, &outer);
            println!("smaller of {inner:?} and {outer:?} is {min:?}");
        } // `min` dropped
    } // `inner` dropped
} // `outer` dropped
```

- Prefer data structures that own their contents where possible
- Use smart pointers for interconnected data structures
- Avoid self-referential data structures