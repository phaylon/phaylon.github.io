
# Rust Shenanigans

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

## Guarding Against ZSTs at Compile-Time

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



