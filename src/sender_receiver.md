# Sender-receiver Pattern

The sender-receiver pattern separates the writing and reading ends of
a data structure to a sender and a receiver. The sender and receiver
can be distributed to different tasks so that the reading and writing
can be done concurrently.

The pattern is commonly used in _channels_, where then sending and
receiving ends are sender and receiver.

```rust
let (tx, rx) = flume::bounded(3);  // Create sender and receiver ends.

let producer_task = spawn(async move {
    for n in 0..100 {
        let result = tx.send().await;
        if result.is_err() {
            break;
        }
    }
});

let consumer_task = spawn(async move {
    while let Some(n) = rx.recv().await {
        println!("{}", n);
    }
});

join!(producer_task, consumer_task);
```



## MPMC Channel

The MPMC channel refers to multi-producer multi-consumer channel. The
sender and receiver ends of this channel can be cloned and distribute
to multiple concurrent tasks.

```rust
let (tx, rx) = flume::bounded(3);

let producer_tasks: Vec<_> = (0..n_producers).map(move |p_id| {
    let tx = tx.clone();
    
    spawn(async move {
        for n in 0..100 {
            let result = tx.send().await;
            if result.is_err() {
                break;
            }
        }
    })
})
.collect();

let consumer_tasks: Vec<_> = (0..n_consumers).map(move |c_id| {
    let rx = rx.clone();
    
    spawn(async move {
        while let Some(n) = rx.recv().await {
            println!("{}", n);
        }
    })
})
.collect();
```


## Oneshot Channel

The oneshot channel delivers at most one message. It is used to mark
the completion of another task.

```rust
let (tx, rx) = oneshot::new();

let anticipated_task = spawn(async move {
    sleep(Duration::from_secs(5)).await;
    tx.send(100);
});

let waiting_task = spawn(async move {
    let value = rx.recv().await;
    match value {
        Some(n) => println!("{}", n),
        None => panic!("the sender did not send a value"),
    }
});
```


## Pub-sub or Watched State

The pub-sub or watched state provides one writer and multiple
watchers. The writer updates the state. The update event is propagated
to the watchers. The watchers can receive the notifications late so
that it only reads the most recent update.

```rust
let (tx, rx) = watch::new();

let writer_task = spawn(async move {
    for n in 0..100 {
        sleep(Duration::from_millis(100)).await;
        tx.send(n);
    }
});

let rx1 = rx.clone();

let watcher1_task = spawn(async move {
    while let Some(n) = rx1.recv().await {
        println!("{}", n);
        sleep(Duration::from_millis(200)).await;
    }
});

let rx2 = rx;

let watcher2_task = spawn(async move {
    while let Some(n) = rx2.recv().await {
        println!("{}", n);
        sleep(Duration::from_millis(200)).await;
    }
});
```

## CRDT

The CRDT is similar to watched state, but supports multiple concurrent
operators. The Bounded Counter is taken for example here. It provides
`increase()`, `decrease()` operations, and multiple receivers to learn
the updates.

```rust
let (increaser, decrease, rx) = bcounter::new();

let increase_task = spawn(async move {
    for n in 0..100 {
        sleep(Duration::from_millis(100)).await;
        let value = rng.gen();  // generates a random value
        increaser.inc(value);
    }
});

let decrease_task = spawn(async move {
    for n in 0..100 {
        sleep(Duration::from_millis(99)).await;
        let value = rng.gen();
        decreaser.dec(value);
    }
});


let watcher_task = spawn(async move {
    while let Some(count) = rx.recv().await {
        println!("current count: {}", count);
    }
});
```

## Implementation of Sender-receiver Pattern

This pattern provides two kinds of operations, namely the `send()` and
`recv()`. Each operation generates a future that eventually send or
receive a value. In order to implement this feature, the polling
functions for `send()` and `recv()` are provided.
