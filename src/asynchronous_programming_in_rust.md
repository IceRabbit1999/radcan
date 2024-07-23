<!-- tags: book, test -->

<!-- collect
{
  "data": {
    "type": "json",
    "keywords": ["json", "collect"]
  }
}
-->

# Asynchronous Programming in Rust

* Concurrency is about dealing with a lot of things at the same time, Concurrency is about working smarter
* Parallelism is about doing a lot of things at the same time, Parallelism is a way of throwing more resources atthe problem
* threads can be a means to perform tasks in parallel, but they can also be a means to achieve concurrency
* asynchronous programming: the way a programming language or library abstracts over concurrent operations, and how we as users of a language or library use that abstraction to execute tasks concurrently
* the operating system is an abstraction over the hardware
* a system call uses a public API that the operating system provides so that programs we write in userland can communicate with the OS
* memory management unit: translate the virtual address we use in our programs to a physical address
* most modern CPUs are built with an OS in mind, so they provide the hooks and infrastructure that the OS latches onto upon bootup
* cooperative: tasks that yield voluntarily when it can't progress further before another operation has finished
* non-cooperative: tasks that don't necessarily yield voluntarily, the scheduler can stop the task and take control over the CPU even through the task would have been able to do work and progress
* stackful: 
  * each task has its own call stack(similar to the stack used by the os for its threads)
  * stackful tasks can suspend execution at any point in the program as the whole stack is preserved
* stackless:
  * there is not a separate stack for each task, they all run sharing the same call stack
  * a task can't be suspended in the middle of a stack frame
  * they need to store/restore less imformation when switching between tasks
* os threads:
  * simple to understand, easy to use, switching between tasks is reasonably fast, get parallelism for free
  * come with a rather large stack, context switching can be costly, might not switch back to you thread as fast as you'd wish, tightly coupled to an os abstraction
* fibers/green thread:
  * simple to use(look like os threads), context switching is fast, less memory issue, full control over how tasks are scheduled
  * stacks need a way to grow when run out of space, still need to save the CPU state on contextswitch, complicated to support many platforms and/or CPU architectures
  * FFI is very complicated
* callback based approaches: the whole idea is to save a pointer to a set of instructions we want to run later together with whatever state is needed(closure in Rust)
  * easy to implement, no context switching, low memory overhead(in most cases)
  * memory usage grows linearly, hard to reason about, affect almost all aspects of the program, owership can be hard to reason about, sharing state between tasks is difficult, debugging can be difficult
* coroutines: promises and futures, each task is represented as a state machine
* coroutines and async/await:
  * write code the same way you normally would, no context switching, can be implemented in a memory-efficient way, easy to implement for various platforms
  * pre-emption can be hard, need compiler support to leverage all advantages, debugging can be difficult
* is all I/O blocking? The answer is Maybe: os is smart and will cache a lot of information in memory, a syscall requesting that information would simply return immediately
* a bitmask is a way to treat each bit as a switch, or a flag, to indicate that an option is either enabled or disabled
* epoll can notify events in a level-triggered or edge-triggered mode
  * level-triggered: a read event has accurred as long as there is data in the buffer associated with the file handle. can encounter a case where we get multiple notifications on the same event
  * edge-triggered: a read event has occurred when the buffer has changed from having no data to having data, if you don't drain the buffer properly, you will never receive a notification on that file handle again
* CPUs implement an instruction set, the instruction set defines what instructions the CPU can execute and the infrastructure it should provide to programmers
* registers are fixed sized slots the CPU need to temporarily store and operate on data
* the stack is nothing more than a piece of contiguous memory
  * the stack supports simple push/pop instructions on a contiguous part of memory, thst's what makes it fast to use
  * the heap memory is allocated by a memory allocator on demand and can be scattered around in different locations
* async in Rust using a poll-based approach inwhich an asynchronous task will have three phases
  1. poll phase
     * resulting in the task progressing until a point where it can no longer make progress
     * executor: the part of the runtime that polls a future 
  2. wait phase
     * reactor: an event source, registers that a future is waiting for an event to happen and makes sure that it will wake the future when that event is ready
  3. wake phase
     * the event happens and the future is woken up
     * it is now up to the executor that polled the future in step 1 to schecule the future to be polled again and make further progress until it completes or reaches a new pointer where it can not make futher progress and the cycle repeats
* leaf futures: runtimes creates leaf futures, which represent a resource such as a socket
* non-leaf futures: the kind of futures we as users of a runtime write ourselves using the `async` keyword to create a task that can be run on the executor
* a fully working async system in Rust can be divided into three parts:
  * reactor: responsible for notifying about I/O events
  * executor: scheduler
  * future: a task taht can stop and resume at specific points


* step1
  * executor holds a list of futures
  * try to run the future by polling it, and hands it a waker
  * when receiving Pending/Ready, the executor knows it can start polling a different future
* step2
  * reactor stores a copy of waker that the executor passed to the future when it polled it
  * reactor tracks events on that I/O source 
* step3
  * when the reactor gets a notification that an event has happend on one of the tracked sources, it locates the waker associated with that source and calls `Waker::wake` on it
  * this will in turn inform the executor that the future is ready to make progress so it can poll it once more 
* executors and reactors never need to communicate with one another directly since the waker is the same across all executors
* rust only provides what's necessary to model asynchronous operations in the language:
  * a common interface that represents an operation, which will be completed in the future through the `Future` trait
  * an ergonomic way of creating tasks that can be suspended and resumed through `async` and `await`
  * a defined interface to wake up a suspended task through the `Waker` type
* a stackless coroutine is a way of representing a task that can be interrupted and resumed
* async/await: if we tagged the start of each function and each point we wanted to yield control back to the caller with a few keywords and had our state machine generated for us
* all runtimes in Rust need to do at least two things: schedule and drive objects implementing `Future` trait to completion
  * reactor/driver: the part that thracks events we're waiting on and makes sure to wait on notifications from the OS in an efficient manner
  * executor: the part that schedules tasks and polls them to completion
* an executor simply decides who gets time on the CPU to progress and when they get it, it must also call `Future::poll` and advance the state machines to its next state
* a loosely coupled reactor and executor


* the space coroutines need will grow linearly with the number of different variables that need to be stored/restored, Rust optimizes this by implementing coroutines as enums, where each state only holds the data it needs to restore in the next state, this way, the size of a coroutine is not dependent on the total number of variables, it's only dependent on the size of the largest state that needs to be saved/restored. it's an advantage that stackless coroutines have over stackful coroutines when it comes to memory efficiency
* moving a self-referential struct


 

 

 
