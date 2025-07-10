User-Level Thread Library
Overview
This project implements a user-level thread library in C for Linux. The system supports preemption, locks, condition variables, semaphores, scheduling with multi-level priority queues, and timer-based context switching. It provides a self-contained environment for thread management and includes test programs and a design document for demonstration.

Core Components
1. Scheduler
The scheduler selects the highest-priority thread from a set of 128 priority queues. If a thread is scheduled for the first time, stack space is allocated, and its context is set using a long jump. The system keeps a count of threads at each priority level, scanning from highest to lowest to find the next thread to execute.

2. Timer Handler
Each thread is allocated 10ms to execute. When the time expires, a SIGVTALRM signal is triggered and handled by the timer handler. The handler saves the context of the current thread and invokes the scheduler to pick the next one. Timer signals are managed using SA_NOMASK to allow reentrant handling.

3. Increment Stack
This function sets up stack space for threads executing for the first time. It moves the stack pointer forward and never returns. Upon thread exit, it checks for any dependent threads and enqueues them for execution.

4. Timer Control
To avoid race conditions, timer interrupts are disabled (using sigprocmask with SIG_BLOCK) during critical operations and re-enabled afterward.

Thread System Workflow
Execution begins with main() calling t_start, which sets up internal variables, stack space, and the timer.

t_fork creates a new thread, assigns it a priority, and inserts it into the appropriate queue. If it has higher priority, it preempts the current thread.

Scheduler() saves its context and allocates space via sitbottom() for the thread system stack, then continues execution.

Threads execute via increment_stack, which adjusts the stack and calls user-defined functions.

Context switching occurs through explicit yield calls, thread exits, or timer interrupts.

Synchronization Primitives
Condition Variables
Each condition variable has a waiter queue and an associated lock. When a thread calls t_wait, it releases the lock, saves its context, and enters the queue. A t_signal checks the waiter queue and hands off the lock if a waiting thread is present.

Locks
Implemented with a binary variable (0 or 1), locks use busy-waiting for availability. Once acquired, timer interrupts are disabled to prevent context switches in critical sections.

Semaphores
Counting semaphores consist of a value, ID, and waiter queue. Semaphore_P blocks if the count is zero; Semaphore_V either wakes a waiting thread or increments the count.

Thread Lifecycle Management
t_join
Allows threads to wait for the completion of other threads. If the target thread has exited, it returns immediately. Otherwise, the calling thread is blocked.

Thread Exit Tracking
Exited threads are recorded by ID in a dedicated array, allowing t_join to verify their termination status.

Scheduling and Timing Details
Preemption ensures higher-priority threads interrupt lower-priority ones.

Round-robin scheduling is applied among threads with equal priority.

Time slice is set to 10ms, ensuring a balance between responsiveness and scheduler overhead.

Interrupt Handling and Critical Sections
Asynchronous interrupts (timer signals) can corrupt shared data; hence, critical sections disable interrupts.

sigprocmask is used to manage signal blocking and unblocking.

Stack Management
t_start allocates space for system variables.

Scheduler invokes sitbottom, which reserves 50 * 5000 bytes of stack for up to 50 threads.

increment_stack allocates per-thread stack space by shifting the pointer, ensuring previously active thread stacks remain untouched.

Assumptions and Design Considerations
The stack can accommodate up to 50 threads. Exceeding this requires adjusting the allocation in sitbottom.

This design proactively prevents crashes due to stack exhaustion by allocating known limits upfront.

Context Switch Triggers
Context switches occur during:

Preemption by higher-priority threads.

Time slice expiration.

Thread exit.

Yield operations.

Wait on conditions.

Semaphore operations.

Impact of Interrupts
Improper handling of interrupts can corrupt queues or shared states.

Disabling interrupts during critical operations ensures consistent system behavior.

This user-level thread library provides a fully functional and synchronized environment suitable for multi-threaded applications, solving classical problems like Producer-Consumer, Bridge, and Elevator problems through robust primitives and preemptive scheduling.



---------------------------------------------------------------------------


Shreyansh Gupta
Email: guptashreyansh245@gmail.com
