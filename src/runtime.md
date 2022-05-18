# Runtime

The _runtime_ schedules the execution of concurrent tasks in the
system. There are various designs of the runtime. Here we focus on the
design of two mainstream runtimes in Rust ecosystem: tokio and
async-std.

The runtime consists of the following components

- Task queue
- Thread pool
- Executor
- Reactor


## Task

The runtime maintains a list of _tasks_, which can yield the thread
during execution and resume later. A task can be constructed from a
future. For example, tokio library provides `tokio::spawn()` to turn a
future to a standalone task.

```rust
let future = File::open("myfile.txt"); // returns a future, not a file handler
let file = tokio::spawn(future).await;
```


The yielding is signaled by `Pending` returned by the poll
function. To signal the runtime that the task is ready to continue,
the poll function stores the **waker** provided by runtime, and raises
the flag using the waker when the time is appropriate.

## Thread Pool

A runtime can have one or multiple threads to process concurrent
tasks. Each thread runs an executor to execute tasks that are ready to
make progress from a work-stealing queue.

## Executor and Reactor

Executor and reactor are two different approaches to schedule
tasks. An executor is a loop that picks up and poll a task
repeatedly. The reactor, on the other hand, distributes _waikers_ to
each task, and pick up a task when its waiker is notified. They are
applied in different scenarios.

The executor is adopted in the case when a task must be polled
multiple times to make progress. For example, a forwarding task that
moves messages from one worker the the other. This task moves one
message when it is polled. It runs until the messages are depleted.

The reactor is flavored for I/O tasks. For example, the task that a
socket waits for an incoming connection.

Both tokio and async-std adopts the hybrid approach. Each thread runs
an executor in work-stealing style. When there is no active tasks, it
goes into the reactor to pick up awaiken tasks.


![](https://pic1.zhimg.com/v2-a06c7cc6e763add8c60e561e9f8d078f_1440w.jpg?source=172ae18b)
Reference: https://zhuanlan.zhihu.com/p/137353103
