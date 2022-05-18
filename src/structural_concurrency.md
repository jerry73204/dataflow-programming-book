# Structural Concurrency

Structural concurrency is a programming paradim where the the task
lifetime are recursively _scoped_ by another other task.

![](https://miro.medium.com/max/1400/1*lus7pX2Ss7Yt47DzDiEdOA.png)
Reference: https://blog.softwaremill.com/structured-concurrency-and-pure-functions-92dd8ed1a9f2

The concept can be realized by _scoped tasks_, each of which has a
determined lifetime scope. The scoped task is natively supported by
tokio and async-std yet.

In order to implement scoped task, the runtime must carefully handle
the task cancelation.
