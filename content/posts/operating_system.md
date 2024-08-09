---
title: "Concepts about Operating System"
date: 2024-08-09T16:05:32+08:00
draft: false
---

## Four Fundamental OS concepts
### Threads: Execution Context 
- Fully describes program state
- Program Counter, Registers, Execution Flags, Stack
### Address space 
- Set of memory addresses accessible to program (for read or write)  
- May be distinct from memory space of the physical machine (in which case programs operate in a virtual address space)
### Process: an instance of a running program
- Protected Address Space + One or more Threads
### Dual mode operation / Protection
- Only the “system” has the ability to access certain resources
- Combined with translation, isolates programs from each other and the OS from programs
## Thread of Control 
- Thread: single unique execution context
    - Program Counter, Registers, Execution Flags, Stack, Memory State
- A thread is ***executing*** on a processor (core) when it is ***resident*** in the processor registers
- Resident means: Registers hold the root state (context) of the thread:
    - Including program counter (PC) register & currently executing instruction  
        - PC points at next instruction in memory  
        - Instructions stored in memory  
    - Including intermediate values for ongoing computations  
        - Can include actual values (like integers) or pointers to values in memory  
    - Stack pointer holds the address of the top of stack (which is in memory)  
    - The rest is “in memory”  
- A thread is ***suspended*** (not ***executing***) when its state is not loaded (resident) into the processor  
    - Processor state pointing at some other thread  
    - Program counter register is not pointing at next instruction from this thread  
    - Often: a copy of the last value for each register stored in memory  

Assume a single processor. How do we provide the illusion of multiple processors? The answer is **Multiplex** in time. Threads are virtual cores, and it contains Program Counter, Stack Pointer and Registers. Either it is running on a physical core, or it is saved in chunk of memory – called the ***Thread Control Block (TCB)***.

A thread is a single execution sequence that represents *a separately schedulable task*.
- **Single execution sequence**. Each thread executes a sequence of instructions — assignments, conditionals, loops, procedures, and so on — just as in the familiar sequential programming model.
- **Separately schedulable task**. The operating system can run, suspend, or resume a thread at any time.

### How does operating system provide the illusion of an infinite number of processors?
To map an arbitrary set of threads to a fixed set of processors, operating systems include a thread scheduler that can switch between threads that are running and those that are ready but not running.  
Threads thus provide an execution model in which each thread runs on a dedicated virtual processor with unpredictable and variable speed.  
There are many possible interleavings of a program with multiple threads.
### How does the operating system implement the thread abstraction?
- **Per-Thread state and Thread Control Block (TCB)**  
    The operating system needs a data structure to represent a thread’s state; This data structure is called the thread control block (TCB).  
    For every thread the operating system creates, it creates one TCB.  
    The thread control block holds two types of per-thread information: 
    - The state of the computation being performed by the thread.
        - **Stack**: it stores information needed by the nested procedures the thread is currently running.
        - **Copy of Processor registers**: A processor's registers include not only its general-purpose registers for storing intermediate values for ongoing computations, they also include special-purpose registers, such as the instruction pointer and stack pointer. To be able to suspend a thread, run another thread, and later resume the original one, the OS needs a place to store a thread's registers when that thread is not actively running. 
    - Metadata about the thread that is used to manage the thread.  
        **Per-Thread metadata**: Information for managing the thread, including thread ID, scheduling priority, and status.

- **Shared State**  
    As opposed to per-thread state that is allocated for each thread, some state is shared between threads running in the same process or within the operating system kernel. In particular, program code is shared by all threads in a process. Additionally, statically allocated global variables and dynamically allocated heap variables can store information that is accessible to all threads.

### Thread Life Cycle 
- **INIT**  
    Thread creation puts a thread into its INIT state and allocates and initializes perthread data structures. Once that is done, thread creation code puts the thread into the READY state by adding the thread to the *ready list*. The ready list is the set of runnable threads that are waiting their turn to use a processor.
- **READY**  
    A thread in the READY state is available to be run but is not currently running. Its TCB is on the *ready list*, and the values of its registers are stored in its TCB. At any time, the scheduler can cause a thread to transition from READY to RUNNING by copying its register values from its TCB to a processor’s registers.
- **RUNNING**   
    A thread in the RUNNING state is running on a processor. At this time, its register values are stored on the processor rather than in the TCB. A RUNNING thread can transition to the READY state in two ways:
    - The scheduler can preempt a running thread and move it to the READY state by: (1) saving the thread’s registers to its TCB and (2) switching the processor to run the next thread on the ready list.
    - A running thread can voluntarily relinquish the processor and go from RUNNING to READY by calling yield (e.g., thread_yield in the thread library).
- **WAITING**  
    A thread in the WAITING state is waiting for some event. Whereas the scheduler can move a thread in the READY state to the RUNNING state, a thread in the WAITING state cannot run until some action by another thread moves it from WAITING to READY. The TCB is stored on the *waiting list* of some synchronization variable associated with the event. When the required event occurs, the operating system moves the TCB from the synchronization variable’s waiting list to the scheduler’s ready list, transitioning the thread.
from WAITING to READY.
- **FINISHED**  
    A thread in the FINISHED state never runs again. The system can free some or all of its state for other uses, though it may keep some remnants of the thread in the FINISHED state for a time by putting the TCB on a *finished list*.

### Alternative Abstractions
- **Asynchronous I/O and event-driven programming**. Asynchronous I/O and events allow a single-threaded program to cope with high-latency I/O devices by overlapping I/O with processing and other I/O.
- **Data parallel programming**. With data parallel programming, all processors perform the same instructions in parallel on different parts of a data set.

### Synchronizing Access to Shared Objects
- When cooperating threads share state, writing correct multi-threaded programs becomes much more difficult.
    Because sequential model of reasoning does not work in programs with cooperating threads, for three reasons:
    - Program execution depends on the possible interleavings of threads’ access to shared state.
    - Program execution can be nondeterministic.
    - Compilers and processor hardware can reorder instructions.
- A better approach is to:
    - structure the program to facilitate reasoning about concurrency, and
    - use a set of standard synchronization primitives to control access to shared state.

#### Challenges
- **Race Condition**  
    A *race condition* occurs when the behavior of a program depends on the interleaving of
operations of different threads. In effect, the threads run a race between their operations,
and the results of the program execution depends on who wins the race.  
    Reasoning about even simple programs with race conditions can be difficult. To appreciate
this, we now look at three extremely simple multi-threaded programs.

- **Atomic Operations**  
    When we disassembled the code in last example, we could reason about atomic
operations, indivisible operations that cannot be interleaved with or split by other
operations.  
    On most modern architectures, a load or store of a 32-bit word from or to memory is an
atomic operation. So, the previous analysis reasoned about interleaving of atomic loads
and stores to memory.
    Conversely, a load or store is not always an atomic operation. Depending on the hardware
implementation, if two threads store the value of a 64-bit floating point register to a memory
address, the final result might be the first value, the second value, or a mix of the two.

#### Critical Section Problem  
In operating systems, the *critical section problem* is a classic synchronization issue that arises when multiple processes or threads need to access shared resources concurrently. The primary goal is to ensure that *only one process or thread* is executing its critical section at any given time to prevent race conditions and ensure data consistency.  
- **Critical Section**
    A critical section is a part of a program **where the process accesses shared resources**, such as variables, data structures, or hardware devices. If multiple processes execute their critical sections simultaneously, it can lead to inconsistent or corrupted data.
- **Requirements** for Solving the Critical Section Problem  
    To handle the critical section problem, a solution must satisfy the following conditions:  
    1. **Mutual Exclusion**: Only one process can enter the critical section at a time. If a process is executing in its critical section, no other process should be allowed to enter its critical section.
    2. **Progress**: If no process is in the critical section, and some processes wish to enter their critical sections, then only the processes not in their remainder section (i.e., the part of the program outside the critical section) should participate in the decision of which process enters the critical section next. This decision should not be postponed indefinitely.
    3. **Bounded Waiting**: There must be a limit on the number of times other processes are allowed to enter their critical sections after a process has made a request to enter its critical section and before the requested process is granted access. This ensures that no process waits indefinitely (no starvation).
#### A Generalization  
There is a classical solution for the Critical Section Problem, called Peterson’s algorithm, which works with any fixed number of N threads. This is a software-based solution for two processes that uses flags and a turn variable to ensure mutual exclusion, progress, and bounded waiting.

#### A Better Solution: Structuring Shared Objects
We write shared objects that use synchronization objects to coordinate different threads’ access to shared state.  
Suppose, for example, we had a primitive called a lock that only one thread at a time can own.  
- *Shared Objects* OR *Monitor*
    Objects that can be accessed safely by multiple threads. All shared state in a program — including variables allocated on the heap (e.g., objects allocated with malloc or new) and static, global variables — should be encapsulated in one or more shared objects.
    Shared objects hide the details of synchronizing the actions of multiple threads behind a clean interface. The threads using shared objects need only understand the interface; they do not need to know how the shared object internally handles synchronization.
    Since shared objects encapsulate the program’s shared state, the main loop code that defines a thread’s high-level actions need not concern itself with synchronization details. The programming model thus looks very similar to that for single-threaded code.
- How Shared Objects are *Implemented*
    They're implemented in layers from top to bottom:
    1. **Shared Object layer**: As in standard object-oriented programming, shared objects define application-specific logic and hide internal implementation details. Externally, they appear to have the same interface as you would define for a single-threaded program. (e.g., *Bounded Buffer*, *Barrier*)
    2. **Synchronization variable layer**. Rather than implementing shared objects directly with carefully interleaved atomic loads and stores, shared objects include synchronization variables as member variables. *Synchronization variables*, stored in memory just like any other object, can be included in any data structure. (e.g., *Semaphores, Locks, Condition Variables*)  
    A *synchronization variable* is a data structure used for coordinating concurrent access to shared state. Both the interface and the implementation of synchronization variables must be carefully designed.  
    Synchronization variables coordinate access to *state variables*, which are just the normal member variables of an object that you are familiar with from single-threaded programming.  
    3. **Atomic instruction layer**. Although the layers above benefit from a simpler programming model, it is not turtles all the way down. Internally, synchronization variables must manage the interleavings of different threads’ actions. Modern implementations build synchronization variables using *atomic read-modify-write instructions*. These *processor-specific instructions* let one thread have temporarily exclusive and atomic access to a memory location while the instruction executes. Typically, the instruction atomically reads a memory location, does some simple arithmetic operation to the value, and stores the result. The hardware **guarantees** that any other thread’s instructions accessing the same memory location will occur either entirely before, or entirely after, the atomic read-modify-write instruction. (e.g., *Interrupt Disable*, *Test-and-Set*)
#### *Multiple types* of *Synchronization Variable*  
- *Lock*  
    A lock is a synchronization variable that provides mutual exclusion — when one thread holds a lock, no other thread can hold it (i.e., other threads are excluded). A program associates each lock with some subset of shared state and requires a thread to hold the lock when accessing that state. Then, only one thread can access the shared state at a time.

    Mutual exclusion greatly simplifies reasoning about programs because a thread can perform an arbitrary set of operations while holding a lock, and those operations appear to be atomic to other threads. In particular, because a lock enforces mutual exclusion and threads must hold the lock to access shared state, no other thread can observe an intermediate state. Other threads can only observe the state left after the lock release.

    It is much easier to reason about interleavings of atomic groups of operations rather than interleavings of individual operations for two reasons. First, there are (obviously) fewer interleavings to consider. Reasoning about interleavings on a coarser-grained basis reduces the sheer number of cases to consider. Second, and more important, we can make each atomic group of operations correspond to the logical structure of the program, which allows us to reason about invariants not specific interleavings.

    In particular, shared objects usually have one lock guarding all of an object’s state. Each public method acquires the lock on entry and releases the lock on exit. Thus, reasoning about a shared class’s code is similar to reasoning about a traditional class’s code: we assume a set of invariants when a public method is called and re-establish those invariants before a public method returns.

    - Case Study: *Thread-Safe Bounded Queue*  
        A *bounded queue* is a queue with a fixed size limit on the number of items stored in the queue. Operating system kernels use bounded queues for managing interprocess communication, TCP and UDP sockets, and I/O requests. Because the kernel runs in a finite physical memory, the kernel must be designed to work properly with finite resources. For example, instead of a simple, infinite buffer between a *producer and a consumer thread*, the kernel will instead use a limited size buffer, or bounded queue.

        A *thread-safe bounded queue* is a type of a bounded queue that is safe to call from multiple concurrent threads, it lets any number of threads safely insert and remove items from the queue.

- *Condition Variables: Waiting for a Change*  
    **Condition variables** provide a way for one thread to wait for another thread to take some action. For example, in the thread-safe queue, rather than returning an error when we try to remove an item from an empty queue, we might wait until the queue is non-empty, and then always return an item.

    Similarly, a web server might wait until a new request arrives; a word processor might wait for a key to be pressed; a weather simulator’s coordinator thread might wait for the worker threads calculating temperatures in each region to finish;

    In all of these cases, we want a thread to wait for some action to change the system state so that the thread can make progress.

    One way for a thread to wait would be to **poll** — to **repeatedly check** the shared state to see if it has changed. Unfortunately, this approach is inefficient: the waiting thread continually loops, or busywaits, consuming processor cycles without making useful progress. Worse, busy-waiting can delay the scheduling of other threads — perhaps exactly the thread for which the looping thread is waiting.

    - **Definition**  
        A *condition variable* is a **synchronization object** that lets a thread efficiently wait for a change to shared state that is protected by a lock. A condition variable has three methods:  
        - **CV::wait(Lock \*lock)**. This call *atomically* *releases the lock and suspends execution* *of the calling thread*, placing the calling thread on the condition variable’s waiting list. Later, when the calling thread is re-enabled, it *re-acquires the lock* before returning from the **wait** call.
        - **CV::signal()**. This call takes one thread off the condition variable’s waiting list and marks it as eligible to run (i.e., it puts the thread on the scheduler’s ready list). If no threads are on the waiting list, **signal** has no effect.
        - **CV::broadcast()**. This call takes all threads off the condition variable’s waiting list and marks them as eligible to run. If no threads are on the waiting list, **broadcast** has no effect.

    - **Design Pattern**  
        The standard design pattern for a shared object is a lock and zero or more condition variables. A method that waits using a condition variable. the calling thread first acquires the lock and can then read and write the shared object’s state variables. To wait until testOnSharedState succeeds, the thread calls **wait** on the shared object’s condition variable cv. This atomically puts the thread on the waiting list and releases the lock, allowing other threads to enter the critical section. Once the waiting thread is signaled, it re-acquires the lock and returns from wait. The monitor can then safely test the state variables to see if testOnSharedState succeeds. If so, the monitor performs its tasks, releases the lock, and returns.

        Whenever a thread changes the shared object’s state in a way that enables a waiting thread to make progress, the thread must **signal** the waiting thread using the condition variable.

        A thread waiting on a condition variable must inspect the object’s state in a *loop*. The condition variable’s **wait** method releases the lock (to let other threads change the state of interest) and then re-acquires the lock (to check that state again).

        Similarly, the only reason for a thread to **signal** (or broadcast) is that it has just *changed the shared state* in a way that may be of interest to a waiting thread.

    - Case Study: *Blocking Bounded Queue*  
        We can use condition variables to implement a *blocking bounded queue*, one where a thread trying to remove an item from an empty queue will wait until an item is available, and a thread trying to put an item into a full queue will wait until there is room.

        Now, however, we can atomically release the lock and wait if there is no room in insert or no item in remove. Before returning, insert signals on itemAdded since a thread waiting in remove may now be able to proceed; similarly, remove signals on itemRemoved before it returns.

#### Three Case Studies
- Reader/Writer Lock  
    - **Definition**  
    *A readers/writers lock* (RWLock) protects shared data. However, it makes the following optimization. To maximize performance, an RWLock allows multiple “reader” threads to simultaneously access the shared data. Any number of threads can safely read shared data at the same time, as long as no thread is modifying the data. However, only one “writer” thread may hold the RWLock at any one time. (While a “reader” thread is restricted to only read access, a “writer” thread may read and write the data structure.) When a writer thread holds the RWLock, it may safely modify the data, as the lock guarantees that no other thread (whether reader or writer) may simultaneously hold the lock. The *mutual exclusion* is thus between any writer and any other writer, and between any writer and the set of readers.
    - **Appliance**  
        Reader-writer locks are very commonly used in databases, where they are used to support faster search queries over the *database*, while also supporting less frequent database updates. Another common use is inside the *operating system kernel*, where core data structures are often read by many threads and only infrequently updated.
- Synchronization Barriers  
    With data parallel programming, as we explained in Chapter 4, the computation executes in parallel across a data set, with each thread operating on a different partition of the data. Once all threads have completed their work, they can safely use each other’s results in the next (data parallel) step in the algorithm. MapReduce is an example of data parallel programming, but there are many other systems with the same structure.

    For this to work, we need an efficient way to check whether all n threads have finished their work. This is called a *synchronization barrier*. It has one operation, **checkin**. A thread calls checkin when it has completed its work; no thread may return from checkin until all n threads have checked in. Once all threads have checked in, it is safe to use the results of the previous step.

- FIFO Blocking Bounded Queue  
    Assuming Mesa semantics for condition variables, our implementation of the thread-safe blocking bounded queue in Figure 5.8 does not guarantee freedom from starvation. For example, a thread may call remove and wait in the while loop because the queue is empty. Starvation would occur if every time another thread inserts an item into the queue, a different thread calls remove, acquires the lock, sees that the queue is full, and removes the item before the waiting thread resumes.

    Often, starvation is not a concern. For example, if we have one thread putting items into the queue, and n equivalent worker threads removing items from the queue, it may not matter which of the worker threads goes first. Even if starvation is a concern, as long as calls to insert and remove are infrequent, or the buffer is rarely empty or full, every thread is highly likely to make progress.

    Suppose, however, we do need a thread-safe bounded buffer that does guarantee progress to all threads. We can more formally define the liveness constraint as:

    - **Starvation-freedom**. If a thread waits in insert, then it is guaranteed to proceed after a bounded number of remove calls complete, and vice versa.
    - **First-in-first-out (FIFO)**. A stronger constraint is that the queue is first-in-first-out, or FIFO. The nth thread to acquire the lock in remove retrieves the item inserted by the nth thread to acquire the lock in insert.

#### Hardware Primitives
Either way, the challenge is to atomically modify those data structures.

Therefore, modern implementations use more powerful hardware primitives that let us atomically read, modify, and write pieces of state. We use two hardware primitives:

1. **Disabling interrupts**. On a single processor, we can make a sequence of instructions atomic by disabling interrupts on that single processor.
2. **Atomic read-modify-write instructions**. On a multiprocessor, disabling interrupts is insufficient to provide atomicity. Instead, architectures provide special instructions to atomically read and update a word of memory. These instructions are globally atomic with respect to the instructions on every processor.

- Implementing Uniprocessor Locks by Disabling Interrupts  
On a uniprocessor, any sequence of instructions by one thread appears atomic to other threads if no context switch occurs in the middle of the sequence. So, on a uniprocessor, a thread can make a sequence of actions atomic by disabling interrupts (and refraining from calling thread library functions that can trigger a context switch) during the sequence.

This implementation does provide the mutual exclusion property we need from locks. Some uniprocessor kernels use this simple approach, but it does not suffice as a general implementation for locks. If the code sequence the lock protects runs for a long time, interrupts will be disabled for that long. This will prevent other threads from running, and it will make the system unresponsive to handling user inputs or other real-time tasks. Furthermore, although this approach can work in the kernel where all code is (presumably) carefully crafted and trusted to release the lock quickly, we cannot let untrusted user-level code run with interrupts turned off since a malicious or buggy program could then monopolize the processor.

- Implementing Uniprocessor Queueing Locks  
A more general solution is based on the observation that if the lock is BUSY, there is no point in running the acquiring thread until the lock is free.  instead, we should context switch to the next ready thread.

The implementation briefly disables interrupts to protect the lock’s data structures, but reenables them once a thread has acquired the lock or determined that the lock is BUSY. The Lock implementation shown in Figure 5.15 illustrates this approach. If a lock is BUSY when a thread tries to acquire it, the thread moves its TCB onto the lock’s waiting list. The thread then suspends itself and switches to the next runnable thread. The call to suspend does not return until the thread is put back on the ready list, e.g., until some thread calls Lock::release.

- Implementing Multiprocessor Spinlocks  
On a multiprocessor, however, disabling interrupts is insufficient. Even when interrupts are turned off on one processor, other threads are running  concurrently. Operations by a thread on one processor are interleaved with operations by other threads on other processors.

Since turning off interrupts is insufficient, most processor architectures provide *atomic read-modify-write instructions* to support synchronization. These instructions can read a value from a memory location to a register, modify the value, and write the modified value to memory atomically with respect to all instructions on other processors.

As an example, some architectures provide a *test-and-set* instruction, which atomically reads a value from memory to a register and writes the value 1 to that memory location.
```c
class SpinLock {
    private:
        int value = 0; // 0=FREE; 1 = BUSY
    public:
        void acquire(){
            while(test_and_set(&value)) // while BUSY
            ; // spin
        }

        void release(){
            value = 0;
            memory_barrier();
        }
}
```
Figure 5.16 implements a lock using test_and_set. This lock is called a spinlock because a thread waiting for a BUSY lock “spins” (busy-waits) in a tight loop until some other lock releases the lock. This approach is inefficient if locks are held for long periods. However, for locks that are only held for short periods (i.e., less time than a context switch would take), spinlocks make sense.

- Implementing Multiprocessor Queueing Locks  
Often, we need to support critical sections of *varying length*. For example, we may want a general solution that does not make assumptions about the running time of methods that hold locks.

We cannot completely eliminate *busy-waiting* on a multiprocessor, but we can *minimize* it. As we mentioned, the scheduler ready list needs a spinlock. The scheduler holds this spinlock for only a few instructions; further, if the ready list spinlock is BUSY, there is no point in trying to switch to a different thread, as that would require access to the ready list.

To reduce *contention* on the ready list spinlock, we use a *separate spinlock* to guard access to each lock’s internal state. Once a thread holds the lock’s spinlock, the thread can inspect and update the lock’s state. If the lock is FREE, the thread sets the value and releases its spinlock. If the lock is BUSY, more work is needed: we need to put the current thread on the waiting list for the lock, suspend the current thread, and switch to a new thread.

Careful sequencing is needed, however, as shown in Figure 5.17. To suspend a thread on a multiprocessor, we need to first *disable interrupts* to ensure the thread is not preempted while holding the ready list spinlock. We then acquire the ready list spinlock, and only then is it safe to release the lock’s spinlock and switch to a new thread. The ready list spinlock is released by the next thread to run. Otherwise, a different thread on another processor
might put the waiting thread back on the ready list (and start it running) before the waiting thread has completed its context switch.

Later, when the lock is released, if any threads are waiting for the lock, one of them is moved off the lock’s waiting list to the scheduler’s ready list.

- Implementing Condition Variables  
We can implement condition variables using a similar approach to the one used to implement locks, with one simplification: since the lock is held whenever the wait, signal, or broadcast is called, we already have mutually exclusive access to the condition wait queue. As with locks, care is needed to prevent a waiting thread from being put back on the ready list until it has completed its context switch; we can accomplish this by *acquiring the scheduler spinlock* before we release the monitor lock. Another thread may acquire the monitor lock and start to signal the waiting thread, but it will not be able to complete the signal until the scheduler lock is released immediately after the context switch.  

#### Semaphores  
Semaphores were introduced by Dijkstra to provide synchronization in the THE operating system, which (among other advances) explored structured ways of using concurrency in operating system design.  
Semaphores are defined as follows:  
    * A semaphore has a non-negative value.
    * When a semaphore is created, its value can be initialized to any non-negative integer.
    * Semaphore::P() waits until the value is positive. Then, it atomically decrements value by 1 and returns. 
    * Semaphore::V() atomically increments the value by 1. If any threads are waiting in P, one is enabled, so that its call to P succeeds at decrementing the value and returns.
    * No other operations are allowed on a semaphore; in particular, no thread can directly read the current value of the semaphore.

Semaphores are signaling mechanisms that can be used to control access to the critical section. A *binary semaphore* (similar to a mutex) can be used to provide mutual exclusion, while *counting semaphores* can manage access to multiple instances of a resource.

To use a semaphore as a mutual exclusion lock, initialize it to 1. Then, Semaphore::P is equivalent to Lock::acquire, and Semaphore::V is equivalent to Lock::release.

Semaphore P and V can be set up to behave similarly. Typically (but not always), you initialize the semaphore to 0. Then, each call to Semaphore::P waits for the corresponding thread to call V. If the V is called first, then P returns immediately.

The difficulty comes when trying to coordinate shared state (needing mutual exclusion) with general waiting. From a distance, Semaphore::P is similar to CV::wait(&lock) and Semaphore::V is similar to CV::signal. However, there are important differences. First, CV::wait(&lock) atomically releases the monitor lock, so that you can safely check the shared object’s state and then atomically suspend execution.

By contrast, Semaphore::P does not release an associated mutual exclusion lock. Typically, the lock is released before the call to P; otherwise, no other thread can access the shared state until the thread resumes. The programmer must carefully construct the program to work properly in this case. Second, whereas a condition variable is stateless, a semaphore has a value. If no threads are waiting, a call to CV::signal has no effect, while a call to Semaphore::V increments the value. This causes the next call to Semaphore::P to proceed without blocking.

First, using separate lock and condition variable classes makes code more selfdocumenting and easier to read. As the quote from Dijkstra notes, two different
abstractions are needed, and code is clearer when the role of each synchronization variable is made clear through explicit typing.

Second, a stateless condition variable bound to a lock is a better abstraction for generalized waiting than a semaphore. By binding a condition variable to a lock, we can conveniently wait on any arbitrary predicate on an object’s state. In contrast, semaphores rely on the programmer to carefully map the object’s state to the semaphore’s value so that a decision to wait or proceed in P can be made entirely based on the value, without holding a lock or examining the rest of the shared object’s state.

Although we do not recommend writing new code with semaphores, code based on semaphores is not uncommon, especially in operating systems. So, it is important to understand the semantics of semaphores and be able to read and understand semaphorebased code written by others.

### Deadlock
A challenge to constructing complex multi-threaded programs is the possibility of deadlock. A deadlock is *a cycle of waiting* among a set of threads, where each thread *waits* for some other thread in the cycle to take some action.

Deadlock can occur in many different situations, but one of the simplest is mutually recursive locking:
```
// Thread A
lock1.acquire();
lock2.acquire();
lock2.release();
lock1.release();
// Thread B
lock2.acquire();
lock1.acquire();
lock1.release();
lock2.release();
```
We can also get into deadlock with two locks and a condition variable, shown below:  
```
// Thread A
lock1.acquire();
...
lock2.acquire();
while (need to wait) {
cv.wait(&lock2);
}
...
lock2.release();
...
lock1.release();
// Thread B
lock1.acquire();
...
lock2.acquire();
...
cv.signal();
lock2.release();
...
lock1.release();
```

Suppose we have two bounded buffers, where one thread puts a request into one buffer, and gets a response out of the other. Deadlock can result if another thread does the reverse.   
```
// Thread A
buffer1.put();
buffer1.put();
...
buffer2.get();
buffer2.get();
// Thread B
buffer2.put();
buffer2.put();
...
buffer1.get();
buffer1.get();
```
If the buffers are almost full, both threads will need to wait for there to be room, and so neither will be able to reach the point where they can pull data out of the other buffer to allow the other thread to make progress.

The scarce resource leading to deadlock can even be a chopstick. The Dining Philosophers problem is a classic illustration of both the challenges and solutions to deadlock; There is a round table with n plates alternating with n chopsticks around the circle. A philosopher sitting at a plate requires two
chopsticks to eat. Suppose that each philosopher proceeds by picking up the chopstick on the left, picking up the chopstick on the right, eating, and then putting down both chopsticks. If every philosopher follows this approach, there can be a deadlock: each philosopher takes the chopstick on the left but can be stuck waiting for the philosopher on the right to release the chopstick.

Note that mutually recursive locking is equivalent to Dining Philosophers with n = 2.

#### Deadlock vs. Starvation
Deadlock and starvation are both liveness concerns. In starvation, a thread fails to make progress for an indefinite period of time. Deadlock is a form of starvation but with the *stronger condition*: a group of threads forms a cycle where none of the threads make progress because each thread is waiting for some other thread in the cycle to take action. Thus, deadlock implies starvation (literally, for the dining philosophers), but starvation does not imply deadlock.

#### Necessary Conditions for Deadlock
There are four necessary conditions for deadlock to occur. Knowing these conditions is useful for designing solutions: if you can prevent any one of these conditions, then you can eliminate the possibility of deadlock.
1. **Bounded resources**. There are a finite number of threads that can simultaneously use a resource.
2. **No preemption**. Once a thread acquires a resource, its ownership cannot be revoked until the thread acts to release it.
3. **Wait while holding**. A thread holds one resource while waiting for another. This condition is sometimes called *multiple independent requests* because it occurs when a thread first acquires one resource and then tries to acquire another.
4. **Circular waiting**. There is a set of waiting threads such that each thread is waiting for a resource held by another.

The four conditions are necessary but not sufficient for deadlock. When there are multiple instances of a type of resource, there can be a cycle of waiting without deadlock because a thread not in the cycle may return resources that enable a waiting thread to proceed.

#### Preventing Deadlock
For an arbitrary program, preventing deadlock can take one of three approaches:
1. **Exploit or limit the behavior of the program.** Often, we can change the behavior of a program to prevent one of the four necessary conditions for deadlock, and thereby eliminate the possibility of deadlock. 
2. **Predict the future.** If we can know what threads may or will do, then we can avoid deadlock by having threads wait (e.g., thread 2 can wait at step 2 above) before they would head into a possible deadlock.
3. **Detect and recover.** Another alternative is to allow threads to recover or “undo” actions that take a system into a deadlock

if a system is structured to prevent at least one of the conditions, then the system cannot deadlock. Considering these conditions in the context of a given system often points to a viable deadlock prevention strategy. Below, we discuss some commonly used approaches.
- **Bounded resources**: Provide sufficient resources. One way to ensure deadlock freedom is to arrange for sufficient resources to satisfy all threads’ demands. A simple example would be to add a single chopstick to the middle of the table in Dining Philosophers; that is enough to eliminate the possibility of deadlock. As another example, thread implementations often reserve space in the TCB for the thread to be inserted into a waiting list or the ready list. While it would be theoretically possible to dynamically allocate space for the list entry only when it is needed, that could open up the chance that the
system would run out of memory at exactly the wrong time, leading to deadlock.  
- **No preemption**: Preempt resources. Another technique is to allow the runtime system to forcibly reclaim resources held by a thread. For example, an operating system can preempt a page of memory from a running process by copying it to disk in order to prevent applications from deadlocking as they acquire memory pages.
- **Wait while holding**: Release lock when calling out of module. For nested modules, each of which has its own lock, waiting on a condition variable in an inner module can lead to a nested waiting deadlock. One solution is to restructure a module’s code so that no locks are held when calling other modules.
- **Circular waiting**: Lock ordering. An approach used in many systems is to identify an ordering among locks and only acquire locks in that order. Likewise, we can eliminate deadlock among the dining philosophers if — instead of always picking up the chopstick on the left and then the one on the right — the philosophers number the chopsticks from 1 to n and always pick up the lower-numbered chopstick before the higher-numbered one.

#### The Banker’s Algorithm for Avoiding Deadlock
A general technique to eliminate wait-while-holding is to wait until all needed resources are available and then to acquire them atomically at the beginning of an operation, rather than incrementally as the operation proceeds. 

Of course, a thread may not know exactly which resources it will need to complete its work, but it can still acquire all resources that it might need. Consider an operating system for mobile phones where memory is constrained and cannot be preempted by copying it to disk. Rather than having applications request additional memory as needed, we might instead have each application state its maximum memory needs and allocate that much memory when it starts.

Dijkstra developed the Banker’s Algorithm as a way to improve on the performance of acquire-all. Although few systems use it in its full generality, we include the discussion because simplified versions of the algorithm are common. The Banker’s Algorithm also sheds light on the distinction between safe and unsafe states and how the occurrence of deadlocks often depends on a system’s workload and sequence of operations.

In the Banker’s Algorithm, a thread states its maximum resource requirements when it begins a task, but it then acquires and releases those resources incrementally as the task runs. The runtime system *delays* granting some requests to ensure that the system never deadlocks.

The insight behind the algorithm is that a system that may deadlock will not necessarily do so: for some interleavings of requests it will deadlock, but for others it will not. By *delaying* when some resource requests are processed, a system can avoid interleavings that could lead to deadlock.

- Key Concepts
    1. Processes and Resources:  
        - The system consists of a fixed number of processes and resource types.
        - Each process has a maximum resource demand and a current allocation.
    2. Data Structures:
        - Available: A vector that indicates the number of available instances of each resource type.
        - Max: A matrix that defines the maximum demand of each process for each resource type.
        - Allocation: A matrix that indicates the current allocation of resources to each process.
        - Need: A matrix derived from Max and Allocation that indicates the remaining resource needs of each process (Need[i][j] = Max[i][j] - Allocation[i][j]).
- Algorithm Steps
    1. Initialization:
        - Initialize the Available, Max, Allocation, and Need matrices.
    2. Request Resources:
        - When a process requests resources, check if the request can be granted.
        - Calculate if the system will remain in a safe state if the request is granted.
        - A safe state means there is at least one sequence of processes that can be fully executed to completion without leading to a deadlock.
    3. Safety Check:
        - Simulate the allocation of requested resources.
        - Check if the resulting state is safe by performing the safety algorithm.
- Safety Algorithm
    1. Work and Finish Initialization:
        - Work: A copy of the Available vector.
        - Finish: An array to indicate if a process can finish (initialized to false for all processes).
    2. Find a Process:
        - Find an unfinished process whose needs can be met with the available resources (i.e., Need[i] ≤ Work).
        - If found, assume the process finishes, release its resources, and update Work and Finish.
    3. Check State:
        - If all processes can finish (all Finish[i] are true), the state is safe.
        - If any process cannot be finished, the state is unsafe, and the resource request is denied.

#### Detecting and Recovering From Deadlocks
Rather than preventing deadlocks, some systems allow deadlocks to occur and recover from them when they arise.

Why allow deadlocks to occur at all? Sometimes, it is difficult or expensive to enforce sufficient structure on the system’s data and workloads to guarantee that deadlock will never occur. If deadlocks are rare, why pay the overhead in the common case to prevent them?

- Recovering From Deadlocks  
we need some systematic way to recover when some required resource is unavailable. Two widely used approaches have been developed to deal with this issue:
    - **Proceed without the resource**. Web services are often designed to be resilient to resource unavailability. A rule of thumb for the web is that a significant fraction of a web site’s customers will give up and go elsewhere if the site’s latency becomes too long, for whatever reason. Whether the problem is a hardware failure, software failure, or deadlock, does not really matter. The web site needs to be designed to quickly respond back to the user, regardless of the type of problem. Because deadlocks are rare and hard to test for, this requires coding discipline to handle error conditions systematically throughout the program.
    - **Transactions: rollback and retry**. A more general technique is used by transactions; transactions provide a safe mechanism for revoking resources assigned to a thread. We discuss transactions in detail in Chapter 14; they are widely used in both databases and file systems. For deadlock recovery, transactions provide two important services:  
        1. *Thread rollback*. Transactions ensure that revoking locks from a thread does not leave the system’s objects in an inconsistent state. Instead, we rollback, or undo, the deadlocked thread’s actions to a clean state. To fix the deadlock, we can choose one or more victim threads, stop them, undo their actions, and let other threads proceed.
        2. *Thread restarting*. Once the deadlock is broken and other threads have completed some or all of their work, the victim thread is restarted. When these threads complete, the system behaves as if the victim threads never caused a deadlock but, instead, just had their executions delayed.

- Detecting Deadlock  
Once we have a general way to recover from a deadlock, we need a way to tell if a deadlock has occurred, so we know when to trigger the recovery. An important
consideration is that the detection mechanism can be conservative: it can trigger the repair if we might be in a deadlock state. This approach risks a false positive where a nondeadlocked thread is incorrectly classified as deadlocked. Depending on the overhead of the repair operation, it can sometimes be more efficient to use a simpler mechanism for detection even if that leads to the occasional false positive.  
There are various ways to identify deadlocks more precisely.  
    - System resource-allocation graph  
        If there are several resources and only one thread can hold each resource at a time (e.g., one printer, one keyboard, and one audio speaker or several mutual exclusion locks), then we can detect a deadlock by analyzing a simple graph. In the graph, each thread and each resource is represented by a node. There is a directed edge (i) from a resource to a thread if the resource is owned by the thread and (ii) from a thread to a resource if the thread is waiting for the resource. There is a deadlock if and only if there is a cycle in such a graph.  
        If there are multiple instances of some resources, then we represent a resource with k interchangeable instances (e.g., k equivalent printers) as a node with k connection points. Now, a cycle is a necessary but not sufficient condition for deadlock.
    - a variation of Dijkstra’s Banker’s Algorithm  
        A variation of Dijkstra's Banker's Algorithm used to detect deadlock, rather than to prevent it, is the Deadlock Detection Algorithm. Unlike the Banker's Algorithm, which is proactive and avoids unsafe states by carefully allocating resources, the Deadlock Detection Algorithm is reactive and periodically checks the system for deadlocks after processes have already requested and allocated resources.
        - Key Concepts  
            1. Processes and Resources:  
                - The system has a fixed number of processes and resource types.
                - Processes request and hold resources without the preventive measures of the Banker's Algorithm.
            2. Data Structures:  
                - Available: A vector indicating the number of available instances of each resource type.
                - Allocation: A matrix that represents the number of resources of each type currently allocated to each process.
                - Request: A matrix that indicates the current requests of each process for additional resources.
        - Algorithm Steps
            1. Initialization:
                - Create and maintain the Available, Allocation, and Request matrices.
            2. Detection Process:
                - Periodically or when a resource request cannot be immediately granted, the system runs the Deadlock Detection Algorithm.
                - The algorithm checks if there is a set of processes that are deadlocked, i.e., waiting for resources that cannot be provided due to circular dependencies.
            3. Work and Finish Initialization:
                - Work: Initialize Work as a copy of the Available vector.
                - Finish: Initialize a Finish array to false for all processes. A process is marked true if it can finish and release its resources.
            4. Find a Process:
                - Look for a process P_i such that Request[i] ≤ Work. If found, assume that P_i can finish, release its resources, and update Work by adding the resources in Allocation[i].
                - Mark Finish[i] as true, indicating that P_i can complete.
            5. Detection:
                - Repeat the above step until no such process P_i can be found.
                - If there are processes that cannot finish (i.e., Finish[i] is still false for some i), these processes are deadlocked.

### Scheduling

#### Uniprocessor Scheduling  
We start by considering one processor, generalizing to multiprocessor scheduling policies in the next section. We begin with three simple policies — first-in-first-out, shortest-job-first, and round robin — as a way of illustrating scheduling concepts. Each approach has its own the strengths and weaknesses, and most resource allocation systems (whether for processors, memory, network or disk) combine aspects of all three. At the end of the discussion, we will show how the different approaches can be synthesized into a more practical and complete processor scheduler.

Before proceeding, we need to define a few terms. A *workload* is a set of tasks for some system to perform, along with when each task arrives and how long each task takes to complete. In other words, the workload defines the input to a scheduling algorithm. Given a workload, a processor scheduler decides when each task is to be assigned the processor.

We are interested in scheduling algorithms that work well across a wide variety of environments, because workloads will vary quite a bit from system to system and user to user. Some tasks are *compute-bound* and only use the processor. Others, such as a compiler or a web browser, mix I/O and computation. Still others, such as a BitTorrent download, are *I/O-bound*, spending most of their time waiting for I/O and only brief periods computing. In the discussion, we start with very simple compute-bound workloads and then generalize to include mixtures of different types of tasks as we proceed.

Some of the policies we outline are the best possible policy on a particular metric and workload, and some are the worst possible policy. When discussing optimality and pessimality, we are only comparing to policies that are *work-conserving*. A scheduler is work-conserving if it never leaves the processor idle if there is work to do. Obviously, a trivially poor policy has the processor sit idle for long periods when there are tasks in the ready list.

- **First-In-First-Out (FIFO)**  
Perhaps the simplest scheduling algorithm possible is first-in-first-out (FIFO): do each task in the order in which it arrives. (FIFO is sometimes also called first-come-first-served, or FCFS.) When we start working on a task, we keep running it until it finishes. FIFO minimizes overhead, switching between tasks only when each one completes. Because it minimizes overhead, if we have a fixed number of tasks, and those tasks only need the processor, FIFO will have the best throughput: it will complete the most tasks the most quickly. And as we mentioned, FIFO appears to be the definition of fairness — every task patiently waits its turn.  
Unfortunately, FIFO has a weakness. If a task with very little work to do happens to land in line behind a task that takes a very long time, then the system will seem very inefficient. If the first task in the queue takes one second, and the next four arrive an instant later, but each only needs a millisecond of the processor, then they will all need to wait until the first one finishes. The average response time will be over a second, but the optimal average response time is much less than that. In fact, if we ignore switching overhead, there are some workloads where FIFO is literally the worst possible policy for average response time.  
- **Shortest Job First (SJF)**  
Suppose we could know how much time each task needed at the processor. (In general, we will not know, so this is not meant as a practical policy! Rather, we use it as a thought experiment; later on, we will see how to approximate SJF in practice.) If we always schedule the task that has the least remaining work to do, that will minimize average response time. (For this reason, some call SJF shortest-remaining-time-first or SRTF.)  
To see that SJF is optimal, consider a hypothetical alternative policy that is not SJF, but that we think might be optimal. Because the alternative is not SJF, at some point it will choose to run a task that is longer than something else in the queue. If we now switch the order of tasks, keeping everything the same, but doing the shorter task first, we will reduce the average response time. Thus, any alternative to SJF cannot be optimal.  
If a long task is the first to arrive, it will be scheduled (if we are work-conserving). When a short task arrives a bit later, the scheduler will preempt the current task, and start the shorter one. The remaining short tasks will be processed in order of arrival, followed by finishing the long task.  
What counts as “shortest” is the remaining time left on the task, not its original length. If we are one nanosecond away from finishing an hour-long task, we will minimize average response time by staying with that task, rather than preempting it for a minute long task that just arrived on the ready list. Of course, if they both arrive at about the same time, doing the minute long task first will dramatically improve average response time.  
Worse, SJF can suffer from starvation and frequent context switches. If enough short tasks arrive, long tasks may never complete. Whenever a new task on the ready list is shorter than the remaining time left on the currently scheduled task, the scheduler will switch to the new task. If this keeps happening indefinitely, a long task may never finish.  
- **Round Robin**
A policy that addresses starvation is to schedule tasks in a round robin fashion. With Round Robin, tasks take turns running on the processor for a limited period of time. The scheduler assigns the processor to the first task in the ready list, setting a timer interrupt for some delay, called the *time quantum*. At the end of the quantum, if the task has not completed, the task is preempted and the processor is given to the next task in the ready list. The preempted task is put back on the ready list where it can wait its next turn. With Round Robin, there is no possibility that a task will starve — it will eventually reach the front of the queue and get its time quantum.  
Of course, we need to pick the time quantum carefully. One consideration is overhead: if we have too short a time quantum, the processor will spend all of its time switching and getting very little useful work done. If we pick too long a time quantum, tasks will have to wait a long time until they get a turn.  
One way of viewing Round Robin is as a compromise between FIFO and SJF. At one extreme, if the time quantum is infinite (or at least, longer than the longest task), Round Robin behaves exactly the same as FIFO. Each task runs to completion and then yields the processor to the next in line. At the other extreme, suppose it was possible to switch between tasks with zero overhead, so we could choose a time quantum of a single instruction. With fine-grained time slicing, tasks would finish in the order of length, as with SJF, but slower: a task A will complete within a factor of n of when it would have under SJF, where n is the maximum number of other runnable tasks.  
Unfortunately, Round Robin has some weaknesses. Round Robin will rotate through the tasks, doing a bit of each, finishing them all at roughly the same time. This is nearly the worst possible scheduling policy for this workload! Time slicing added overhead without any benefit. consider what SJF does on this workload. SJF schedules tasks in exactly the same order as FIFO. The first task that arrives will be assigned the processor, and as soon as it executes a single instruction, it will have less time remaining than all of the other tasks, and so it will run to completion. Since we know SJF is optimal for average response time, this means that both FIFO and Round Robin are optimal for some workloads and pessimal for others, just different ones in each case.  
Depending on the time quantum, Round Robin can also be quite poor when running a mixture of I/O-bound and compute-bound tasks. I/O-bound tasks often need very short periods on the processor in order to compute the next I/O operation to issue. Any delay to be scheduled onto the processor can lead to system-wide slowdowns. For example, in a text editor, it often takes only a few milliseconds to echo a keystroke to the screen, a delay much faster than human perception. However, if we are sharing the processor between a text editor and several other tasks using Round Robin, the editor must wait several time quanta to be scheduled for each keystroke — with a 100 ms time quantum, this can become annoyingly apparent to the user.  
- **Multi-Level Feedback Queue**  
Most commercial operating systems, including Windows, MacOS, and Linux, use a scheduling algorithm called *multi-level feedback queue (MFQ)*. MFQ is designed to achieve several simultaneous goals:  
    - **Responsiveness**. Run short tasks quickly, as in SJF.
    - **Low Overhead**. Minimize the number of preemptions, as in FIFO, and minimize the time spent making scheduling decisions.
    - **Starvation-Freedom**. All tasks should make progress, as in Round Robin.
    - **Background Tasks**. Defer system maintenance tasks, such as disk defragmentation, so they do not interfere with user work.
    - **Fairness**. Assign (non-background) processes approximately their max-min fair share of the processor.  
As with any real system that must balance several conflicting goals, MFQ does not perfectly achieve any of these goals. Rather, it is intended to be a  reasonable compromise in most real-world cases.  
MFQ is an extension of Round Robin. Instead of only a single queue, MFQ has multiple Round Robin queues, each with a different priority level and time quantum. Tasks at a higher priority level preempt lower priority tasks, while tasks at the same level are scheduled in Round Robin fashion. Further, higher priority levels have shorter time quanta than lower levels.  
Tasks are moved between priority levels to favor short tasks over long ones. A new task enters at the top priority level. Every time the task uses up its time quantum, it drops a level; every time the task yields the processor because it is waiting on I/O, it stays at the same level (or is bumped up a level); and if the task completes it leaves the system.   
A new compute-bound task will start as high priority, but it will quickly exhaust its time quantum and fall to the next lower priority, and then the next. Thus, an I/O-bound task needing only a modest amount of computing will almost always be scheduled quickly, keeping the disk busy. Compute-bound tasks run with a long time quantum to minimize switching overhead while still sharing the processor.  
So far, the algorithm we have described does not achieve starvation freedom or max-min fairness. If there are too many I/O-bound tasks, the compute-bound tasks may receive no time on the processor. To combat this, the MFQ scheduler monitors every process to ensure it is receiving its fair share of the resources. At each level, Linux actually maintains two queues — tasks whose processes have already reached their fair share are only scheduled if all other processes at that level have also received their fair share.  
Periodically, any process receiving less than its fair share will have its tasks *increased* in priority; equally, tasks that receive more than their fair share can be *reduced* in priority.  

#### Multiprocessor Scheduling
- Scheduling Sequential Applications on Multiprocessors  
Consider a server handling a very large number of web requests. A common software architecture for servers is to allocate a separate thread for each user connection. Each thread consults a shared data structure to see which portions of the requested data are cached, and fetches any missing elements from disk. The thread then spools the result out across the network.  
How should the operating system schedule these server threads? Each thread is I/Obound, repeatedly reading or writing data to disk and the network, and therefore makes many small trips through the processor. Some requests may require more computation; to keep average response time low, we will want to favor short tasks.  
A simple approach would be to use a centralized multi-level feedback queue, with a lock to ensure only one processor at a time is reading or modifying the data structure. Each idle processor takes the next task off the MFQ and runs it. As the disk or network finishes requests, threads waiting on I/O are put back on the MFQ and executed by the network processor that becomes idle.  
There are several potential performance problems with this approach:  
    - **Contention for the MFQ lock**. Depending on how much computation each thread does before blocking on I/O, the centralized lock may become a bottleneck,particularly as the number of processors increases.  
    - **Cache Coherence Overhead**. Although only a modest number of instructions are needed for each visit to the MFQ, each processor will need to fetch the current state of the MFQ from the cache of the previous processor to hold the lock. On a single processor, the scheduling data structure is likely to be already loaded into the cache. On a multiprocessor, the data structure will be accessed and modified by different processors in turn, so the most recent version of the data is likely to be cached only by the processor that made the most recent update. Fetching data from a remote cache can take two to three orders of magnitude longer than accessing locally cached data. Since the cache miss delay occurs while holding the MFQ lock, the MFQ lock is held for longer periods and so can become even more of a bottleneck.
    - **Limited Cache Reuse**. If threads run on the first available processor, they are likely to be assigned to a different processor each time they are scheduled. This means that any data needed by the thread is unlikely to be cached on that processor. Of course, some of the thread’s data will have been displaced from the cache during the time it was blocked, but on-chip caches are so large today that much of the thread’s data will remain cached. Worse, the most recent version of the thread’s data is likely to be in a remote cache, requiring even more of a slowdown as the remote data is fetched into the local cache.  
Commercial operating systems such as Linux use a per-processor data structure: a separate copy of the multi-level feedback queue for each processor.  
Each processor uses *affinity scheduling*: once a thread is scheduled on a processor, it is returned to the same processor when it is re-scheduled, maximizing cache reuse. Each processor looks at its own copy of the queue for new work to do; this can mean that some processors can idle while others have work waiting to be done. Rebalancing occurs only if the queue lengths are persistent enough to compensate for the time to reload the cache for the migrated threads. Because rebalancing is possible, the per-processor data structures must still be protected by locks, but in the common case the next processor to use the data will be the last one to have written it, minimizing cache coherence overhead and lock contention.  
- Scheduling Parallel Applications  
A different set of challenges occurs when scheduling parallel applications onto a
multiprocessor. There is often a natural decomposition of a parallel application onto a set of
processors. For example, an image processing application may divide the image up into
equal size chunks, assigning one to each processor. While the application could divide the
image into many more chunks than processors, this comes at a cost in efficiency: less
cache reuse and more communication to coordinate work at the boundary between each
chunk.  
If there are multiple applications running at the same time, the application may receive
fewer or more processors than it expected or started with. New applications can start up,
acquiring processing resources. Other applications may complete, releasing resources.
Even without multiple applications, the operating system itself will have system tasks to run
from time to time, disrupting the mapping of parallel work onto a fixed number of
processors.  
    - **Oblivious Scheduling**
    One might imagine that the scheduling algorithms we have already discussed can take care of these cases. Each thread is time sliced onto the available processors; if two or more applications create more threads in aggregate than processors, multi-level feedback will ensure that each thread makes progress and receives a fair share of the processor. This is often called *oblivious scheduling*, as the operating system scheduler operates without knowledge of the intent of the parallel application — each thread is scheduled as a completely independent entity.  
    Unfortunately, several problems can occur with oblivious scheduling on multiprocessors:  
        - **Bulk synchronous delay**. A common design pattern in parallel programs is to split work into roughly equal sized chunks; once all the chunks finish, the processors synchronize at a barrier before communicating their results to the next stage of the computation. This bulk synchronous parallelism is easy to manage — each processor works independently, sharing its results only with the next stage in the computation. At each step, the computation is limited by the slowest processor to complete that step. If a processor is preempted, its work will be delayed, stalling the remaining processors until the last one is scheduled. Even if one of the waiting processors picks up the preempted task, a single preemption can delay the entire computation by a factor of two, and possibly even more with cache effects. Since the application does not know that a processor was preempted, it cannot adapt its decomposition for the available number of processors, so each step is similarly delayed until the processor is returned.
        - **Producer-consumer delay**. Some parallel applications use a producer-consumer
        design pattern, where the results of one thread are fed to the next thread, and the
        output of that thread is fed onward, as in Figure 7.9. Preempting a thread in the middle
        of a producer-consumer chain can stall all of the processors in the chain.
        - **Critical path delay**. More generally, parallel programs have a critical path — the
        minimum sequence of steps for the application to compute its result. Figure 7.10
        illustrates the critical path for a fork-join parallel program. Work off the critical path can
        occur in parallel, but its precise scheduling is less important. Preempting a thread on
        the critical path, however, will slow down the end result. Although the application
        programmer may know which parts of the computation are on the critical path, with
        oblivious scheduling, the operating system will not; it will be equally likely to preempt a
        thread on the critical path as off.
        - **Preemption of lock holder**. Many parallel programs use locks and condition variables
        for synchronizing their parallel execution. Often, to reduce the cost of acquiring locks,
        parallel programs will use a “spin-then-wait” strategy — if a lock is busy, the waiting
        thread spin-waits briefly for it to be released, and if the lock is still busy, it blocks and
        looks for other work to do. This can reduce overhead in the common case that the lock
        is held for only short periods of time. With oblivious scheduling, however, the lock
        holder can be preempted — other tasks will spin-then-wait until the lock holder is rescheduled,
        increasing overhead.
        - **I/O**. Many parallel applications do I/O, and this can cause problems if the operating
        system scheduler is oblivious to the application decomposition into parallel work. If a
        read or write request blocks in the kernel, the thread blocks as well. To reuse the
        processor while the thread is waiting, the application program must have created more
        threads than processors, so that the scheduler can have an extra one to run in place
        of the blocked thread. However, if the thread does not block (e.g., on a file read when
        the file is cached in memory), that means that the scheduler has more threads than
        processors, and so needs to do time slicing to multiplex threads onto processors —
        causing all of the problems we have listed above.
    - **Gang Scheduling**  
    One possible approach to some of these issues is to schedule all of the tasks of a program
    together. This is called *gang scheduling*. The application picks some decomposition of
    work into some number of threads, and those threads run either together or not at all. If the
    operating system needs to schedule a different application, if there are insufficient idle
    resources, it preempts all of the processors of an application to make room. Figure 7.11
    illustrates an example of gang scheduling.  
    Because of the value of gang scheduling, commercial operating systems, such as Linux,
    Windows, and MacOS, have mechanisms for dedicating a set of processors to a single
    application. This is often appropriate on a server dedicated to a single primary use, such as
    a database needing precise control over thread assignment. The application can pin each
    thread to a specific processor and (with the appropriate permissions) mark it to run with
    high priority. The system reserves a small subset of the processors to run other
    applications, multiplexed in the normal way but without interfering with the primary
    application.
    It is usually more efficient to run two parallel programs
    each with half the number of processors, than to time slice the two programs, each *gang*
    scheduled onto all of the processors. Allocating different processors to different tasks is
    called *space sharing*, to differentiate it from time sharing, or time slicing — allocating a
    single processor among multiple tasks by alternating in time when each is scheduled onto
    the processor. Space sharing on a multiprocessor is also more efficient in that it minimizes
    processor context switches: as long as the operating system has not changed the
    allocation, the processors do not even need to be time sliced.
    - **Scheduler Activations**  
    How does the application know how many processors to use if the number
    changes over time?  
    A solution, recently added to Windows, is to make the assignment and re-assignment of
    processors to applications visible to applications. Applications are given an execution
    context, or scheduler activation, on each processor assigned to the application; the
    application is informed explicitly, via an upcall, whenever a processor is added to its
    allocation or taken away. Blocking on an I/O request also causes an *upcall* to allow the
    application to *repurpose* the processor while the thread is waiting for I/O.

#### Real-Time Scheduling
On some systems, the operating system scheduler must account for process **deadlines**.
For example, the sensor and control software to manage an airplane’s flight path must be
executed in a timely fashion, if it is to be useful at all. Similarly, the software to control antilock
brakes or anti-skid traction control on an automobile must occur at a precise time if it
is to be effective. In a less life critical domain, when playing a movie on a computer, the
next frame must be rendered in time or the user will perceive the video quality as poor.

These systems have *real-time constraints*: computation that must be completed by a
*deadline* if it is to have value.

How do we design a scheduler to ensure deadlines are met?

There are three widely used techniques for increasing the likelihood that threads meet their
deadlines. These approaches are also useful whenever timeliness matters without a strict
deadline, e.g., to ensure responsiveness of a user interface.  
- **Over-provisioning**. A simple step is to ensure that the real-time tasks, in aggregate,
use only a fraction of the system’s processing power. This way, the real-time tasks will
be scheduled quickly, without having to wait for higher-priority, compute-intensive
tasks.  
- **Earliest deadline first**. Careful choice of the scheduling policy can also help meet
deadlines. If you have a pile of homework to do, neither shortest job first nor round
robin will ensure that the assignment due tomorrow gets done in time. Instead, realtime
schedulers, mimicking real life, use a policy called *earliest deadline first* (EDF).
EDF sorts tasks by their deadline and performs them in that order. If it is possible to
schedule the required work to meet their deadlines, and the tasks only need the
processor (and not I/O, locks or other resources), EDF will ensure that all tasks are
done in time.  
- **Priority donation**. Another problem can occur through the interaction of shared data
structures, priorities, and deadlines. Suppose we have three tasks, each with a
different priority level. The real-time task runs at the highest priority, and it has
sufficient processing resources to meet its deadline, with some time to spare.
However, the three tasks also access a shared data structure, protected by a lock.  
Suppose the low priority acquires the lock to modify the data structure, but it is then
preempted by the medium priority task. The relative priorities imply that we should run
the medium priority task first, even though the low priority task is in the middle of a
critical section. Next, suppose the real-time task preempts the medium task and
proceeds to access the shared data structure. It will find the lock busy and wait.
Normally, the wait would be short, and the real-time task would be able to meet its
deadline despite the delay. However, in this case, when the high priority task waits for
the lock, the scheduler will pick the medium priority task to run next, causing an
indefinite delay. This is called priority inversion; it can occur whenever a high priority
task must wait for a lower priority task to complete its work.  
A commonly used solution, implemented in most commercial operating systems, is
called priority donation: when a high priority task waits on a shared lock, it temporarily
donates its priority to the task holding the lock. This allows the low priority task to be
scheduled to complete the critical section, at which point its priority reverts to its
original state, and the processor is re-assigned to the high priority, waiting, task.

#### Queueing Theory
- **Definitions**
Because queueing theory is concerned with the root causes of system performance, and
not just its observable effects, we need to introduce a bit more terminology. A simple
abstract queueing system is illustrated by Figure 7.16. In any queueing system, tasks
arrive, wait their turn, get service, and leave. If tasks arrive faster than they can be
serviced, the queue grows. The queue shrinks when the service rate exceeds the arrival
rate.
    - **Server**. A server is anything that performs tasks. A web server is obviously a server,
    performing web requests, but so is the processor on a client machine, since it
    executes application tasks. The cashier at a supermarket and a waiter in a restaurant
    are also servers.
    - **Queueing delay (W) and number of tasks queued (Q)**. The queueing delay, or wait
    time, is the total time a task must wait to be scheduled. In a time slicing system, a task
    might need to wait multiple times for the same server to complete its task; in this case
    the queueing delay includes all of the time a task spends waiting until it is completed.
    - **Service time (S)**. The service time S, or execution time, is the time to complete a task
    assuming no waiting.
    - **Response time (R)**. The response time is the queueing delay (how long you wait in
    line) plus the service time (how long it takes once you get to the front of the line).  
        `R = W + S`
    - **Arrival rate (λ) and arrival process**. The arrival rate λ is the average rate at which
    new tasks arrive.  
    More generally, the arrival process describes when tasks arrive including both the
    average arrival rate and the pattern of those arrivals such as whether arrivals are
    bursty or spread evenly over time.
    - **Service rate (μ)**. The service rate μ is the number of tasks the server can complete
    per unit of time when there is work to do. Notice that the service rate μ is the inverse of
    the service time S.
    - **Utilization (U)**. The utilization is the fraction of time the server is busy (0 ≤ U ≤ 1). In a
    work-conserving system, utilization is determined by the ratio of the average arrival
    rate to the service rate:  
        ```
        U = λ / μ if λ < μ
        = 1 if λ ≥ μ
        ```
    - **Throughput (X)**. Throughput is the number of tasks processed by the system per unit
    of time. When the system is busy, the server processes tasks at the rate of μ, so we
    have:  
        `X = U μ`
    - **Number of tasks in the system (N)**. The average number of tasks in the system is
    just the number queued plus the number receiving service:  
        `N = Q + U`
- **Little’s Law**
*Little’s Law* is a theorem proved by John Little in 1961 that applies to any *stable system*
where the arrival rate matches the departure rate. It defines a very general relationship
between the average throughput, response time, and the number of tasks in the system:  
`N = X R`

Although this relationship is simple and intuitive, it is powerful because the “system” can be
anything with arriving and departing tasks, provided the system is stable — regardless of
the arrival process, number of servers, or queueing order.


## Memory Management

### Address translation 
Address translation is a simple concept, but it turns out to be incredibly powerful. What can
an operating system do with address translation? This is only a partial list:  
- **Process isolation.** As we discussed in Chapter 2, protecting the operating system
kernel and other applications against buggy or malicious code requires the ability to
limit memory references by applications. Likewise, address translation can be used by
applications to construct safe execution sandboxes for third party extensions.
- **Interprocess communication.** Often processes need to coordinate with each other,
and an efficient way to do that is to have the processes share a common memory
region.
- **Shared code segments.** Instances of the same program can share the program’s
instructions, reducing their memory footprint and making the processor cache more
efficient. Likewise, different programs can share common libraries.
- **Program initialization.** Using address translation, we can start a program running
before all of its code is loaded into memory from disk.
- **Efficient dynamic memory allocation.** As a process grows its heap, or as a thread
grows its stack, we can use address translation to trap to the kernel to allocate
memory for those purposes only as needed.
- **Cache management.** As we will explain in the next chapter, the operating system can
arrange how programs are positioned in physical memory to improve cache efficiency,
through a system called page coloring.
- **Memory mapped files.** A convenient and efficient abstraction for many applications is
to map files into the address space, so that the contents of the file can be directly
referenced with program instructions.
- **Efficient I/O.** Server operating systems are often limited by the rate at which they can
transfer data to and from the disk and the network. Address translation enables data to
be safely transferred directly between user-mode applications and I/O devices.
- **Virtual memory.** The operating system can provide applications the abstraction of
more memory than is physically present on a given computer.
- **Persistent data structures.** The operating system can provide the abstraction of a
persistent region of memory, where changes to the data structures in that region
survive program and system crashes.


#### Definition
The translator takes each instruction and data memory reference generated by
a process, checks whether the address is legal, and converts it to a physical memory
address that can be used to fetch or store instructions or data. The data itself — whatever
is stored in memory — is returned as is; it is not transformed in any way. The translation is
usually implemented in hardware, and the operating system kernel configures the
hardware to accomplish its aims.  
Given that a number of different implementations are possible, how should we evaluate the
alternatives? Here are some goals we might want out of a translation box; the design we
end up with will depend on how we balance among these various goals.
- Memory protection
- Memory Sharing
- Flexible memory placement
- Sparse addresses
- Runtime lookup efficiency
- Compact translation tables
- Portability
We will end up with a fairly complex address translation mechanism, and so our discussion
will start with the simplest possible mechanisms and add functionality only as needed. It
will be helpful during the discussion for you to keep in mind the two views of memory: the
process sees its own memory, using its own addresses. We will call these *virtual
addresses*, because they do not necessarily correspond to any physical reality. By
contrast, to the memory system, there are only *physical addresses* — real locations in
memory. From the memory system perspective, it is given physical addresses and it does
lookups and stores values. The translation mechanism converts between the two views:
from a virtual address to a physical memory address.

#### Towards Flexible Address Translation
Our discussion of hardware address translation is divided into two steps. First, we put the
issue of lookup efficiency aside, and instead consider how best to achieve the other goals
listed above: flexible memory assignment, space efficiency, fine-grained protection and
sharing, and so forth. Once we have the features we want, we will then add mechanisms to
gain back lookup efficiency.  
In Chapter 2, we illustrated the notion of hardware memory protection using the simplest
hardware imaginable: base and bounds. The translation box consists of two extra registers
per process. The base register specifies the start of the process’s region of physical
memory; the bound register specifies the extent of that region. If the base register is added
to every address generated by the program, then we no longer need a relocating loader —
the virtual addresses of the program start from 0 and go to bound, and the physical
addresses start from base and go to base + bound. Since physical memory can contain several processes, the kernel
*resets* the contents of the base and bounds registers on *each process context switch* to the
appropriate values for that process.

Base and bounds translation is both simple and fast, but it lacks many of the features
needed to support modern programs. Base and bounds translation supports only coarsegrained
protection at the level of the entire process; it is not possible to prevent a program
from overwriting its own code, for example. It is also difficult to share regions of memory
between two processes. Since the memory for a process needs to be contiguous,
supporting dynamic memory regions, such as for heaps, thread stacks, or memory mapped
files, becomes difficult to impossible.

##### Segmented Memory
Many of the limitations of base and bounds translation can be remedied with a small
change: instead of keeping only a single pair of base and bounds registers per process,
the hardware can support an array of pairs of base and bounds registers, for each process.
This is called *segmentation*. Each entry in the array controls a portion, or segment, of the
virtual address space. The physical memory for each segment is stored contiguously, but
different segments can be stored at different locations. The *high order bits* of the virtual address are used to *index into* the
array; the rest of the address is then treated as above — *added to the base* and checked
against the bound stored at that index. In addition, the operating system can assign
different segments different permissions, e.g., to allow execute-only access to code and
read-write access to data. Although four segments are shown in the figure, in general the
number of segments is determined by the number of bits for the segment number that are
set aside in the virtual address.  

program memory is no longer a single contiguous region, but instead it is a set of regions.
Each different segment
starts at a new segment boundary. For example, code and data are not immediately
adjacent to each other in either the virtual or physical address space.

What happens if a program branches into or tries to load data from one of these gaps? The
hardware will generate an exception, trapping into the operating system kernel. On UNIX
systems, this is still called a segmentation fault, that is, a reference outside of a legal
segment of memory.

Although simple to implement and manage, segmented memory is both remarkably
powerful and widely used. With segments, the operating system can allow
processes to share some regions of memory while keeping other regions protected.

Likewise, shared library routines, such as a graphics library, can be placed into a segment
and shared between processes. This is frequently done in modern operating systems with dynamically
linked libraries.

We can also use segments for interprocess communication, if processes are given read
and write permission to the same segment.

As a final example of the power of segments, they enable the efficient management of
dynamically allocated memory. When an operating system reuses memory or disk space
that had previously been used, it must first zero out the contents of the memory or disk.
Otherwise, private data from one application could inadvertently leak into another,
potentially malicious, application.

Over time, as processes are created and finish, physical memory will
be divided into regions that are in use and regions that are not, that is, available to be
allocated to a new process. These free regions will be of varying sizes. When we create a
new segment, we will need to find a free spot for it. Should we put it in the smallest open
region where it will fit? The largest open region?

However we choose to place new segments, as more memory becomes allocated, the
operating system may reach a point where there is enough free space for a new segment,
but the free space is not contiguous. This is called *external fragmentation*. The operating
system is free to compact memory to make room without affecting applications, because
virtual addresses are unchanged when we relocate a segment in physical memory. Even
so, compaction can be costly in terms of processor overhead: a typical server configuration
would take roughly a second to compact its memory.

All this becomes even more complex when memory segments can grow. How much
memory should we set aside for a program’s heap? If we put the heap segment in a part of
physical memory with lots of room, then we will have wasted memory if that program turns
out to need only a small heap. If we do the opposite — put the heap segment in a small
chunk of physical memory — then we will need to copy it somewhere else if it changes
size.

##### Paged Memory
An alternative to segmented memory is *paged memory*. With paging, memory is allocated
in fixed-sized chunks called *page frames*. Address translation is similar to how it works with
segmentation. Instead of a segment table whose entries contain pointers to variable-sized
segments, there is a *page table* for each process whose entries contain pointers to page
frames. Because page frames are *fixed-sized and a power of two*, the page table entries
only need to provide the *upper bits* of the page frame address, so they are more compact.
There is no need for a “bound” on the offset; the entire page in physical memory is
allocated as a unit.

What will seem odd, and perhaps cool, about paging is that while a program thinks of its
memory as linear, in fact its memory can be, and usually is, scattered throughout physical
memory in a kind of abstract mosaic. The processor will execute one instruction after
another using virtual addresses; its virtual addresses are still linear. However, the
instruction located at the end of a page will be located in a completely different region of
physical memory from the next instruction at the start of the next page. Data structures will
appear to be contiguous using virtual addresses, but a large matrix may be scattered
across many physical page frames.

Paging addresses the principal limitation of segmentation: *free-space allocation* is very
straightforward. The operating system can represent physical memory as a bit map, with
each bit representing a physical page frame that is either free or in use. Finding a free
frame is just a matter of finding an empty bit.

Sharing memory between processes is also convenient: we need to set the page table
entry for each process sharing a page to point to the same physical page frame. For a
large shared region that spans multiple page frames, such as a shared library, this may
require setting up a number of page table entries. Since we need to know when to release
memory when a process finishes, shared memory requires some extra bookkeeping to
keep track of whether the shared page is still in use. The data structure for this is called a
*core map*; it records information about each physical page frame such as which page table
entries point to it.

Page tables allow other features to be added. For example, we can start a program
running before all of its code and data are loaded into memory. Initially, the operating
system marks all of the page table entries for a new process as invalid; as pages are
brought in from disk, it marks those pages as read-only (for code pages) or read-write (for
data pages). Once the first few pages are in memory, however, the operating system can
start execution of the program in user-mode, while the kernel continues to transfer the rest
of the program’s code in the background. As the program starts up, if it happens to jump to
a location that has not been loaded yet, the hardware will cause an exception, and the
kernel can stall the program until that page is available. Further, the compiler can
reorganize the program executable for more efficient startup, by coalescing the initialization
pages into a few pages at the start of the program, thus overlapping initialization and
loading the program from disk.

A *downside* of paging is that while the management of physical memory becomes simpler,
the management of the virtual address space becomes more challenging. Compilers
typically expect the execution stack to be contiguous (in virtual addresses) and of arbitrary
size; each new procedure call assumes the memory for the stack is available. Likewise, the
runtime library for dynamic memory allocation typically expects a contiguous heap. In a
single-threaded process, we can place the stack and heap at opposite ends of the virtual
address space, and have them grow towards each other, as shown in Figure 8.5. However,
with multiple threads per process, we need multiple thread stacks, each with room to grow.

This becomes even more of an issue with 64-bit virtual address spaces. The size of the
page table is proportional to the size of the virtual address space, not to the size of
physical memory. The more sparse the virtual address space, the more overhead is
needed for the page table. Most of the entries will be invalid, representing parts of the
virtual address space that are not in use, but physical memory is still needed for all of
those page table entries.

We can reduce the space taken up by the page table by choosing a larger page frame.
How big should a page frame be? A larger page frame can waste space if a process does
not use all of the memory inside the frame. This is called *internal fragmentation*. Fixed-size
chunks are easier to allocate, but waste space if the entire chunk is not used.

##### Multi-Level Translation
Almost all multi-level address translation systems use paging as the lowest level of the
tree. The main differences between systems are in how they reach the page table at the
leaf of the tree — whether using segments plus paging, or multiple levels of paging, or
segments plus multiple levels of paging. 
- Paged Segmentation  
Let us start a system with only two levels of a tree. With paged segmentation, memory is
segmented, but instead of each segment table entry pointing directly to a contiguous
region of physical memory, each segment table entry points to a page table, which in turn
points to the memory backing that segment. The segment table entry “bound” describes
the page table length, that is, the length of the segment in pages. Because paging is used
at the lowest level, all segment lengths are some multiple of the page size.

Although segment tables are sometimes stored in special hardware registers, the page
tables for each segment are quite a bit larger in aggregate, and so they are normally stored
in physical memory. To keep the memory allocator simple, the maximum segment size is
usually chosen to allow the page table for each segment to be a small multiple of the page
size.
##### Multi-Level Paging
A nearly equivalent approach to paged segmentation is to use multiple levels of page
tables. The top-level page table contains entries, each of which
points to a second-level page table whose entries are pointers to page tables.
##### Multi-Level Paged Segmentation
We can combine these two approaches by using a segmented memory where each
segment is managed by a multi-level page table.


#### Towards Efficient Address Translation
At this point, you should be getting a bit antsy. After all, most of the hardware mechanisms
we have described involve at least two and possibly as many as four memory extra
references, on each instruction, before we even reach the intended physical memory
location! It should seem completely impractical for a processor to do several memory
lookups on every instruction fetch, and even more that for every instruction that loads or
stores data.

In this section, we will discuss how to improve address translation performance without
changing its logical behavior. In other words, despite the optimization, every virtual address
is translated to exactly the same physical memory location, and every permission
exception causes a trap, exactly as would have occurred without the performance
optimization.

For this, we will use a *cache*, a copy of some data that can be accessed more quickly than
the original.

##### Translation Lookaside Buffers
If the two
instructions are on the same page in the virtual address space, then they will be on the
same page in physical memory. The processor will just repeat the same work — the table
walk will be exactly the same, and again for the next instruction, and the next after that.

A translation lookaside buffer (TLB) is a *small hardware table* containing the results of
recent address translations. Each entry in the TLB maps a virtual page to a physical page:

```
TLB entry = {
    virtual page number,
    physical page frame number,
    access permissions
}
```

Instead of finding the relevant entry by a multi-level lookup or by hashing, the TLB
hardware (typically) checks all of the entries simultaneously against the virtual page. If
there is a match, the processor uses that entry to form the physical address, skipping the
rest of the steps of address translation. This is called a TLB hit. On a *TLB hit*, the hardware
still needs to check permissions, in case, for example, the program attempts to write to a
code-only page or the operating system needs to trap on a store instruction to a copy-onwrite
page.

A *TLB miss* occurs if none of the entries in the TLB match. In this case, the hardware does
the full address translation in the way we described above. When the address translation
completes, the physical page is used to form the physical address, and the translation is
installed in an entry in the TLB, replacing one of the existing entries. Typically, the replaced
entry will be one that has not been used recently.

To be useful, the TLB lookup needs to be much
more rapid than doing a full address translation; thus, the TLB table entries are
implemented in very fast, on-chip static memory, situated near the processor. In fact, to
keep lookups rapid, many systems now include multiple levels of TLB. In general, the
smaller the memory, the faster the lookup. So, the first level TLB is small and close to the
processor. If the first level TLB does not contain the translation, a
larger second level TLB is consulted, and the full translation is only invoked if the
translation misses both levels.

##### Superpages
One way to improve the TLB hit rate is using a concept called *superpages*. A superpage is
a set of contiguous pages in physical memory that map a contiguous region of virtual
memory, where the pages are aligned so that they share the same high-order (superpage)
address.

Superpages complicate operating system memory allocation by requiring the system to
allocate chunks of memory in different sizes. However, the upside is that a superpage can
drastically reduce the number of TLB entries needed to map large, contiguous regions of
memory. Each entry in the TLB has a flag, signifying whether the entry is a page or a
superpage. For superpages, the TLB matches the superpage number — that is, it ignores
the portion of the virtual address that is the page number within the superpage.

##### Virtually Addressed Caches
Another step to improving the performance of address translation is to include a virtually
addressed cache before the TLB is consulted, as shown in Figure 8.16. A virtually
addressed cache stores a copy of the contents of physical memory, indexed by the virtual
address. When there is a match, the processor can use the data immediately, without
waiting for a TLB lookup or page table translation to generate a physical address, and
without waiting to retrieve the data from main memory. Almost all modern multicore chips
include a small, virtually addressed on-chip cache near each processor core.


##### Physically Addressed Caches
Many processor architectures include a physically addressed cache that is consulted as a
second-level cache after the virtually addressed cache and TLB, but before main memory. Once the physical address of the memory location is
formed from the TLB lookup, the second-level cache is consulted. If there is a match, the
value stored at that location can be returned directly to the processor without the need to
go to main memory.