# 2. Processing and Scaling

## 2.1. Latency

| Operation | Time in 2009  | Time in 2019 |
| -         | -     | -     |
| L1 cache reference    | 0.5 ns    | 1.5 ns    |
| L2 cache reference    | 7 ns      | 6 ns      |
| Mutex lock/unlock     | 25 ns     | 20 ns     |
| Main memory reference | 100 ns    | 100 ns    |
| Read 1 MB sequentially from memory | 250 000 ns | 12 000 ns |
| Read 1 MB sequentially from disk  | 20 000 000 ns | 10 000 000 ns |
| Disk seek             | 10 000 000 ns | 10 000 000 ns |
| Send packet California -> Netherlands -> California | 150 000 000 ns | 150 000 000 ns |

## 2.2. Threads

- Abstract the execution of a sequence of instructions;
- Simplifying, whereas a process abstracts the execution of a program, a thread abstracts the execution of a function;
  - In today's OSs, a process provides an execution environment for more than one thread.
- Threads of a given process may share most resources, except the stack and theprocessor state;
- Thread, like a process, may have 3 states:
  - Ready;
  - Running;
  - Waiting.
- Thread-specific information:
  - its state (process may be blocked waiting for an event);
  - the processo state (including SP and PC);
  - a stack.
- Operations (like below) on threads of the same process are much more efficient than the same operations on processes:
  - creation/termination;
  - switching.

## 2.3. Threads Implementation

- Kernel-Level Threads: implemented directly by the OS:
  - Kernel supports processes with multiple threads (scheduler allocates cores to threads);
  - OS keeps a threads table with information on every thread;
  - All thread management operations (such as thread creation) incur a system call;
- User-Level Threads: implemented by user-space library:
  - Kernel not aware of the existence of threads at user-level: the OS needs not support threads;
  - Threads' library must provide functions for:
    - thread creation/destruction;
    - thread synchronization;
    - yield a core to other threads.
  - Library responsible for thread switching and keeps a thread's table;
  - Wrapper-functions of some system calls that may block, have to be modified (to prevent other threads from blocking).

Hybrid Implementation (m:n):
- idea: multiplex user-level threads on kernel-level threads;
- kernel not aware of the existence of user-level threads. For better results:
  - User-level scheduler should give hints to kernel-level scheduler;
  - Kernel-level scheduler should notify user-level kernel about its decisions.
- Library maps user-level threads to kernel-level threads: number of user-level threads may be much larger than that of kernel-level threads.

## 2.4. Multi-Threaded Server

- Each thread processes a request;
- When one thread blocks on I/O another thread may be scheduled to run in its place.
- Common pattern:
  - One dispatcher thread, which accepts a connection request;
  - Several worker threads, each of which processes all the requests sent in the scope of a single connection.

Bounding threads' resource usage:
- Thread-Pools:
  - Allow to bound the number of threads;
  - Avoid thread creation/destruction overhead (if you use a fixed number of threads or at least a minimum number of threads, that is large enough);
  - Supported by several packages.
- Excessive Thread-Switch Overhead:
  - This arises more often if you use multiple-thread pools;
  - In this case, you may want to bound the number of active threads (ex: semaphore).

## 2.5. Synchronous vs. Asynchronous I/O

Synchronous I/O have 2 modes:
- Blocking: the thread blocks until the operation is completed (write()/send() system calls may return immediately after copying the data to kernel space and enqueueing the output request);
- Non blocking: the thread doesn't block (not even in input operations: the call returns immediately with whatever data is available at the kernel. In Unix all I/O to block devices, files or directories is blocking).

Asynchronous I/O: the system call just enqueues the I/O request and returns immediately:
- Thread may execute while the requested I/O operation is being executed;
- Thread learns about the termination of the I/O operation (either by polling or via event notification (signals in Unix)).

poll()/epoll() and Blocking I/O:
- Scenario: With TCP, servers use one data socket per cconnection/client.
- Question: Can we use fewer threads than data sockets?
- Answer: Yes: use select()/poll()/epoll().
- poll() blocks until:
  - One of the requested events (ex: data input (POLLIN)) occurs;
  - The timeout (in ms) expires.
- Issue: This doesn't work with regular files (shall always poll TRUE for reading and writing). A work-around is to use helper threads for disk I/O.

POSIX Asynchronous I/O
- POSIX.1b specifies several functions for asynchronous I/O;
- The asynchronous I/O operations are controlled by an AIO control block structure.

Asynchronous I/O: Operation Termination
- Problem: How does the user process learn that the operation has terminated?
- Solution: 2 alternatives specified in the sigev_notify, member of the struct sigevent:
  - Polling (SIGEV_NONE): the process can invoke aio_error() -> it returns EINPROGRESS while it has not completed;
  - Notification:
    - Signal (SIGEV_SIGNAL): the signal is specified in field of the struct sigevent or of the struct aiocb argument (process must register the corresponding handler via the sigaction() system call);
    - Function (SIGEV_THREAD): to be executed by a thread created for that purpose.

## 2.6. Event-Driven Server

- Server executes a loop, in which it:
  - waits for events (usually I/O events);
  - processes these events (sequentially, but may be not in order).
- Blocking is avoided by using non-blocking I/O operations.
- Need to keep a FSM for each request: the loop dispatches the event to the appropriate FSM;
- Known as the state machine approach.

Other Scalability Issues:
- Data copying (especially in network protocols):
  - Use buffer descriptors, not simple pointers;
  - Use scatter/gather I/O (readv()/writev()).
- Memory allocation: design your own which can pre-allocate a pool of memory buffers and avoid to free those buffers;
- Concurrency Control: avoid sharing, locking granularity, minimize the duration of critical sections;
- Kernel/Protocol tuning.