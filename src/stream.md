# Stream

## The Polling Function

A stream represents a sequence of asynchronous values. It is similar
to `Future`, but it can yield more multiple values. The stream types
has a poll function that returns the following values.

- `Ready(Some(x))`: A new value `x` is provided.
- `Pending`: The next value is not ready.
- `Ready(None)`: The stream is finished and should not be polled
  later.


`Stream` is a non-std trait providing a `poll_next()` function.

```rust
pub trait Stream {
    type Item;
    fn poll_next(
        self: Pin<&mut Self>, 
        cx: &mut Context<'_>
    ) -> Poll<Option<Self::Item>>;
}
```

### Stream vs. Iterator

Stream is very similar to iterator. Both can be asked for the next
value, but stream may not be ready, while iterator always return a
value. Using `futures::stream::iter()`, any iterator can be converted
to a stream which polling always returns a ready value.

```rust
use futures::stream;

// iterator
let mut iter = 0..1;
assert!(iter.next() == Some(0));
assert!(iter.next() == None);

// stream
let mut stream = stream::iter(0..1);
assert!(iter.next().await == Some(0));
assert!(iter.next().await == None);
```

Users must be cautious that asking the next value from an iterator can
block, but polling a stream should not. It's a pitfall to convert an
iterator that potentially blocks to a stream!
