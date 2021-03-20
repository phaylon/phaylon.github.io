
# Rust Shenanigans

A Warning: These are the result of me playing around and trying to
get Rust to do interesting things. The examples here might be faulty,
full of downsides, or plain wrong.

## Move Detection

```rust
pub struct MoveDetector {
    last: std::cell::Cell<usize>,
}

impl MoveDetector {
    
    pub fn new() -> Self {
        MoveDetector {
            last: std::cell::Cell::new(0),
        }
    }
    
    pub fn has_moved(&self) -> bool {
        (self as *const _ as usize) != self.last.get()
    }
    
    pub fn reset(&self) {
        self.last.set(self as *const _ as usize);
    }
}

fn main() {

    let md = MoveDetector::new();
    assert!(md.has_moved());
    md.reset();
    assert!(!md.has_moved());
    
    let md = Box::new(md);
    assert!(md.has_moved());
    md.reset();
    assert!(!md.has_moved());
}
```

## Guarding against ZSTs at Compile-Time

```rust
trait AssertNonZst: Sized {
    const ZST_DETECT: usize = 1 / std::mem::size_of::<Self>();
    fn assert_non_zst() -> usize { Self::ZST_DETECT + 0 }
}

impl<T> AssertNonZst for T {}

// compiles
fn main() {
    i32::assert_non_zst();
}

// fails to compile
fn main() {
    struct Zst;
    Zst::assert_non_zst();
}
```

## Newtyping with Opaque Types and Type Aliases

```rust
#![feature(min_type_alias_impl_trait)]

trait AddSelf: std::ops::Add<Output = Self> + Sized {}
impl<T> AddSelf for T where T: std::ops::Add<Output = Self> {}

type MyI64 =
    impl std::fmt::Debug
        + std::fmt::Display
        + AddSelf
        + Copy
        + From<i64>
        + Into<i64>;

fn my_i64(value: i64) -> MyI64 { value }

fn main() {

    // compiles
    let a = my_i64(23);
    let b = my_i64(42);
    println!("{} + {} = {}", a, b, a+b);
    
    let x = MyI64::from(2i64 * a.into());
    println!("{}", x);
    
    // fail to compile
    let x = a + 3i64;
    let x = a * a;
}
```
