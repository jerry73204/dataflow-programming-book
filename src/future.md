# Future

## The Polling Function

A future represents an asynchronous computed value that may not be
ready yet. It is a value that can be asked to be whether `Ready` or
`Pending` many times. `Future` is a trait in Rust standard
library. Types with `Future` trait must have a `poll()`, which returns
returns `Ready` or `Pending` immediately.

```rust
pub trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

## Construct `Future` by `poll()` Function

The straightforward method to construct `Future` types can be done by
writing `poll()` functioon manually.


```rust
struct PollTwiceToComplete {
    count: usize,
}

impl Default for PollTwiceToComplete {
    fn default() -> Self {
        Self {
            count: 0,
        }
    }
}

impl Future for PollTwiceToComplete {
    type Output = ();
    
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        if self.count == 2 {
            Poll::Ready(())
        } else {
            self.count += 1;
            Poll::Pending
        }
    }
}
```

## Construct `Future` by Composition of Futures

A future type (with anonymous name) can be construct by a definition
of `async fn` function. Within the body of `fn`, you can write down
the statement `let output = a_future.await;`.

```rust
use async_std::fs::File;

async fn my_future() -> Result<()> {
    let file = File::open("myfile.txt").await?;
    file.write_all("data").await?;
    file.close().await?;
    Ok(())
}
```
