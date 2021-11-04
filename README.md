# priceholder-rs

A threadsafe hashmap type that notifies waiters of state changes.

### Usage

```rust
use std::{thread, time::Duration};

use priceholder::{PriceHolder, ThreadSafe};

fn main() {
    // `PriceHolder`s are generic and can store any types that are `Unsigned`.
    let mut ph: ThreadSafe<u128> = ThreadSafe::new();

    {
        // `ThreadSafe` encapsulates an `Arc`, so the reference count is automatically
        // increased when the price holder is cloned, making it threadsafe.
        let mut ph = ph.clone();
        // Spawn a new thread ...
        thread::spawn(move || {
            // ... that waits for some time ...
            thread::sleep(Duration::from_secs(1));
            // ... then puts a new price in the price holder for BTC.
            let price = 420;
            ph.put_price("BTC".to_string(), price).unwrap();
            println!("Put price: {}", price);
        })
    };

    // Create a waiter that waits for the price of BTC to be updated in the price
    // holder, by blocking execution of the thread.
    let price = ph.next_price("BTC".to_string()).unwrap();
    println!("Received price: {}", price);
    assert_eq!(price, 420);
}
```

Outputs:

```sh
$ cargo run --quiet main.rs
Put price: 420
Received price: 420
```