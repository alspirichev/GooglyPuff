# GooglyPuff

- [GCD Concepts](#gcd-concepts)
- [Queues](#queues)
- [Synchronous vs. Asynchronous](#synchronous-vs-asynchronous)
- [When to use the various queues with async?](#when-to-use-the-various-queues-with-async)
- [Sources](#sources)

**Grand Central Dispatch (GCD)** is a low-level **API** for managing concurrent operations. **GCD** can help you improve your app’s responsiveness by defering computationally expensive tasks to the background. It’s an easier concurrency model to work with than locks and threads.

## GCD Concepts

In iOS a process or application is made up of one or more **threads**. The threads are managed independently by the operating system scheduler. Each thread can execute concurrently but it’s up to the system to decide if this happens and how it happens.

![concurrency](https://github.com/alspirichev/GooglyPuff/blob/master/img/concurrency.png)

**GCD** is built on top of **threads**. Under the hood it manages a **shared thread pool**. With **GCD** you add **blocks** of code or **work items** to dispatch queues and **GCD** decides which thread to execute them on.

Note that **GCD** decides how much parallelism is required based on the system and available system resources. It’s important to note that parallelism *requires* concurrency, but concurrency does *not guarantee* parallelism.

Basically, **concurrency** is about *structure* while **parallelism** is about *execution*.

## Queues

**GCD** provides dispatch queues represented by **DispatchQueue** to manage tasks you submit and execute them in a **FIFO** order guaranteeing that the *first task submitted is the first one started*.

Queues can be either *serial* or *concurrent*:

- **Serial** queues guarantee that **only one** task runs at any given time.
  
  ![serial](https://github.com/alspirichev/GooglyPuff/blob/master/img/serial.png)
  
- **Concurrent** queues allow **multiple** tasks to run at the same time. Tasks are guaranteed to start in the order they were added.
  
  ![Concurrent](https://github.com/alspirichev/GooglyPuff/blob/master/img/concurrent.png)
  
GCD provides **three** main types of queues:

- **Main** queue: runs on the main thread and is a **serial** queue.
  
- **Global** queues: **concurrent** queues that are shared by the whole system. 
  
  There are **four** such queues with different **priorities**: *high, default, low*, and *background*. (The background priority queue is I/O throttled.)
  
- **Custom** queues: queues that you create which can be *serial* or *concurrent*.

When setting up the *global concurrent queues*, you don’t specify the priority **directly**. Instead you specify a **Quality of Service (QoS)** class property.

The **QoS** classes are:

- **User-interactive**.
  
  This represents tasks that need to be done **immediately** in order to provide a nice *user experience*. Use it for UI updates, event handling and small workloads that require low latency. The total amount of work done in this class during the execution of your app should be small. **This should run on the main thread**.
  
- **User-initiated**.
  
  The represents tasks that are initiated from the UI and can be performed **asynchronously**. It should be used when the user is waiting for immediate results, and for tasks required to continue user interaction. This will get mapped into the *high priority global* queue.
  
- **Utility**.
   
   This represents long-running tasks, typically with a user-visible progress indicator. Use it for computations, I/O, networking, continous data feeds and similar tasks. This class is designed to be energy efficient. This will get mapped into the *low priority global* queue.
   
- **Background**.
   
   This represents tasks that the user is **not directly** aware of. Use it for prefetching, maintenance, and other tasks that don’t require user interaction and aren’t time-sensitive. This will get mapped into the *background priority global* queue.
   
## Synchronous vs. Asynchronous

A **synchronous** function returns control to the caller **after the task is completed**.

An **asynchronous** function returns **immediately**, ordering the task to be done but not waiting for it. Thus, an asynchronous function does not block the current thread of execution from proceeding on to the next function.

## When to use the various queues with async?

* **Main Queue**: This is a common choice to update the **UI** after completing work in a task on a concurrent queue. To do this, you’ll code one closure inside another. Targeting the main queue and calling *async* guarantees that this new task will execute sometime after the current method finishes.

* **Global Queue**: This is a common choice to perform **non-UI** work in the background.

* **Custom Serial Queue**: A good choice when you want to perform **background work** serially and track it. This eliminates resource contention since you know only one task at a time is executing. Note that if you need the data from a method, you must inline another closure to retrieve it or consider using *sync*.

## Sources

* [First part](https://www.raywenderlich.com/148513/grand-central-dispatch-tutorial-swift-3-part-1)
* [Second part](https://www.raywenderlich.com/148515/grand-central-dispatch-tutorial-swift-3-part-2)
