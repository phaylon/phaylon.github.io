
# Move Detection

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
