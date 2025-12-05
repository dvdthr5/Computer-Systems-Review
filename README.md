# Computer Systems Review

--- 

## Intro to Computer Systems

### What is a computer system?

A computer system is a series of interconnected parts which produce an expected outcome. This could be a computer, a transistor, an algorithm, etc. 

### Modularization

Modularizatoin is the act of breakking a system down into smaller parts. This allows for each module to have its own desired outcome. The modules can be layered, formed into a heirarchy, or can function on their own. Breaking a larger system down into modules provides many benefits, the main being fault isolation. Having seperate modules prevents an error in one module from affecting the other systems. \

### Abstractions

Abstracting means to hide the nonrelevent informtaion from the user. A good abstraction should provide the user with only information that they will need in order to solve the given task. A leaky abstraction is an abstraction that is not properly built, and reveals the system internals to a user in an unintended way. Good abstractions hide complexity and system internals, providing public users with the bare minimum tools.

--- 


## Unix and Its Abstractions

### System Calls

System calls are the interface between a user level program and the OS kernel. User programs cannot directly preform privelledged operations so they request these services through system calls. Common examples are: 
- Read/Write (IO File) operations -> read(), write(), open(), close()
- Interprocess Communication -> Pipes, Sockets, Kill()
- Memory Management -> mmap(), brk(), shm_open()
- Process Control -> fork(), join(), waitpid(), exec()

### Processes

Processes are the basic units of execution in an operating system. A system can have multiple processes, though can only optimally run the same number of processes as CPU cores in the machine. A core can only do one thing at a time, so when a core is shared by processes the CPU utilizaes **time multiplexing** in order to make it seem like both processes are similtaneously running. 

A process has its own virtaul memeory space, and cannot affect another programs execution, unless they are piped together. 

### Fork Join Parallelism

A beneficial aspect of a process is its ability to fork and spawn new processes. By calling fork() a process will duplicate itself and pass a new PID to the child. Calling fork() will return 0 for the child if the execution was sucessful adn the child's PID to the parent. Similar to threads, once there are multiple processes running, the order of execution is not guaranteed and the scheduler assings the relative ordering of the events. By using wait() a parent that has finished executing can wait to reap it's child which has not finished. If a parent finishes executing and terminates before its child, the child becomes a 'zombie' and is 'adopted' by the init process with PID = 1;

--- 

## Interprocess Communication

### Signals

Signals are asynchronous, single integer containing broadcasts that are used to notify processes of an event. They are non blocking, though interruppting, meaning that a process may be interruppted at any point in its execution to handle the signal. It only carries an integer byte, so any information tah is attempted to be sent with a signal will be dropped and lost.

### Pipes

Pipes are an interprocess communitcation method that allow for unidirectional communication between processes. A pipe is an array buffer with two elements, the read buffer and the write buffer. In order to write into a pipe you first close the read side of the pipe and write into the write side. The pipe will then transfer the write buffer to the read buffer and the other process will be able to read the bytestream. After the bytes are read from the read buffer, they bytes are dropped and can no longer be accessed. There are two types of pipes, named and anonymous. Anonymous pipes are only usable between processes with a familial relation, like a parent and its child. Named pipes are able to communicate between any two processes, so long as they both have permission to write to the same file. Contrary to signals, pipes are blocking, not asynchronous.

### Sockets

Sockets are very similar to pipes, though they allow for bidirectional communication, meaning that there is no read or write side. Any process that connects to a socket can either read or write to the socket. There are two types of sockets, domain sockets(unix sockets) and network sockets(TCP sockets). Domain sockets allow for bidirectional communication between any processes on the same host machine. Contrary, network sockets allow for communication between two processes that are on different host machines, through the network. Similar to pipes, they are blocking, meaning that if trying to read an empty socket, the process/thread will block until some bytes have been read. 

### Shared Memory

One more interprocess communication method is shared memory. The kernel delegates shared memory in a way that makes both processes think it is their memory. This works by mapping each processes virtaul memory to the same physical pages, which allows for the two processes to share address space. This is omptimal for read only data sets, when the overhead of copying becomes too much. Reading a large file from shared is much cheaper than copying it to the write side of a pipe, tehn copying it to the read side, then copying it to the processes virtaul memory. 

--- 

## Threads and Concurrency

### How is a thread different from a process?

A thread is very similar, though also very different from a processs. Both are units of execution, but a thread exists within a process. There can be many threads inside of the same process, and each thread in the process will share the same virtual memory, which is the main difference. Sharing the same memory introduces a new topic, concurrency, which is importnat now that concurrency issues, like data races and ordering violations can exist in a process. 

### Thread Pools

There are many methods of how to use threads efficiently, one being a **thread pool**. A thread pool has a grouop of worker threads that wait around until they are assingned a task. After completion, they go back to waiting. This is an efficient method for things like serers where burst traffic is possible and tasks need to be completed in the order they are received in. Having teh sleeping workers that wake up when needed removes the oerhead of spawning a new worker per task. 

### Concurrency Issues

#### Race Condition

A race condition is when a programs behavior depends on te timing or the ordering of events. Since the scheduler does not have a set schedule for threads and processes, a program with a race condition wouldn't be itempotent and could produce different results when running with the same input. 

#### Data Races

The most prominent race condition is called a data race, and occurs when two threads are trying to access the same address space with at least one of the threads being a write operation and there is no system to guarantee the ordering of the events. This will result in torn reads, where a thread accesses a value which is not correct. 

#### Atomicity Violations

An atomicity violation occurs when the correctness of a program revolves around a multistep operation being indivisible, but the program doesn't guarantee it. Atomicity violations tend to also be data races. 

#### Ordering Violations

An ordering violation occurs when one thread assumes that an event happens before a different event, but the program nor scheduler does not enorce this. 

--- 

## Synchronization

### Locks

The way that we fix concurrency issues and race conditions is with **mutual exclusion**. Locks, also known as mutexs, create **critical sections** in the program which allow only the thread holding the lock the access that section. Locks are a primitive for mutual exclusion and 'protect' variables and memory that needs to be shared. Locks must be aquired to be held, and then released so that another thread can the hold the lock.

There is a concept called lock contention, which is teh time that threads spend blocked and doing nothing becuase they are waiting to aquire a lock that is already held. Having constantly blocked threads is expensive and inefficient, so we use fine grained locks to improve this. 

### Condition Variables

Condition variables are a mechanism that allow threads to coordinate waiting and signaling when a condition over a shared state is met. Conditional variables are always used in realtion to locks to ensure proper mutal exclusion. Waiting on a conditional variable should alwayd be done inside of a loop, as if there are no threads waiting at the exact time the signal is sent, no threads will receive it. 

### Fine Grained Locks

Fine grained locks are when seperate locks are used for each logical entry in a data structure. At a high level this adds more parallelism to a program and reduces the lock contention, but does increase the cmoplexity of certain operations. This would be like having each file that gets written to in a program have its own lock. Dynamically initializing and aquiring locks is a form of fine grained locks. 

### Reader Writer Locks

Conceptually, a reader writer lock is a lock that the thread specifies which action they will be preforming. In the event that it is a read, multiple threads are allowed to hold the lock, as reading is an itempotent function. When a thread wants to write to the critical section, the lock ensures that no other threads are holding the lock. These work especially well when the read load of a file is significantly higher than the write load or when there are 'hot' items that are very frequently accessed. 

### Deadlocks

Deadlocks occur when a set of threads waitins for an event that only another waiting thread can cause. A graph where a process or thread requires to hold a lock that is being held by another thread that also is waiting for another lock that is held by another thread and so on. In order for a deadlock to occur there must be
- Mutual Exclusion: locks can only be held by one thread at a time
- Hold and Wait: Threads that have resrouces can request and wait for more
- No Preemption: Resources cannot be foribly taken
- Circular Wait: A cycle of threads each waiting on the next

In a **Resource Allocation Graph** this looks like a cyclical graph. 

**Starvation** is the term used to describe a program that will wait forever due to a deadlock. 

--- 

## Client Server Model

### TCP / IP STACK

In order from highest to lowest:
- Application Layer: Application's Format + Protocol
- Transport Layer: End to end host communication
- Network Layer: Addressing and routing to the internet
- Link Layer: Frames on a local link
- Physical Layer: Signals in a medium

A **packet** is a unit of data (that is technically in the networking layer)
A TCP Socket is identifiable by its 4 tuple: (Client IP, Client Port, Server IP, Server Port)

### Serialization (Marshalling)

Because not all machines use the same internal language, a processes pointers are local to that process, structs have different layouts, and endainess, computers must serialize their requests. **Serialization** is the process of turning machine depenedent code into machine independent, 'wire-formats' like JSON, CSV, XML, or Binary. 

### Service Oriented Architechture

Service Oriented Architechture is the idea of breaking a system down into microservices. Very similar to modularization, this allows for better scalability, isolated faults, and better collaborative development environments. Each service acts as a client when it talks to other services and as a server when it responds to other services. 

### Remote Procedure Calls

Remote Procedure Calls(RPCs) are an attempt at reducing the programming burden of client server designs. The goal of RPCs is to make calling a service look and seem like calling a function. 

### RPC Process

When a client (service) wants a service, the client:
- Creates a stub
    - Serializes the data
    - Sends the request over the framework
    - waits for response
The framework:
- Forwards the stub to the server
The server:
- Creates a stub
    - Deserializes the data
    - Passes to handler
      Handler (Dev Made Logic):
        - Executues the desired logic
        - Passes output to server stub
    - Serializes Data
    - Sends the response over the framework
Client Stub:
- Deserializes data
- Passes output to client

### RPC Semantics

When sending requests and responses, there is always the possibility that either the request or reponse will be lost. This leads to undefined behavior when the client cannot access the infomormation that it requests.

#### At Least Once

At least once semantics are RPCs that utilize the retires method. We fix this with the retires method. Each request has a timer associated with it, and if the reponse has not been received by the time the timer hits 0, it retries sending the message until the response is received. This is called at least once, becuase it guarantees that the client will receive a response, though it may receive many more than one response. This method is best used for itempotent functions like reads, where the number of calls to the function would not change the result. 

#### At Most Once

At most once semantics are RPCs that use the ID method of tracking requests and responses. The ID method attaches a unique ID to each request and the server will only respond if the ID is a new ID, in other words, if the request is duplicated, it will respond to at most one of the requests.

#### Exactly Once 

Exactly once semantics is the method that ensure that each request receieves only one response, and ensures that each request always receives a response. The issue with this method is that it is technically impossible to implement, as you would need imfinite memory in the limit.

--- 

## Performance

### Build Then Measure

Build then measure is a method that involves building a system end to end and then using benchmarks to measure the performance of the system. After benchmarking, you locate the constraint, or the section that consumes the most resources then fix that constraint. 

### Capacity 

Capacity is the total amount of resources that a system can supply. 

### Useful Work and Overhead

Useful work is the amount of resources that go towards a users direct work. Overhead is the exact opposite, the amount of resources that do not go directly to the users work. Things like context switching, duplicating processes, etc cause expensive over head. 

### Efficiency

Efficiency of a system is determined by useful work divided by the capacity. The higher this percentage the better, as it is the total resources of the system that are directly used for helping the users work. 

### Time Based Metrics

#### Latency

Latency is how long it takes to process a unit of work. The lower the latency of a system, the better. When discussing latency, we generally split it into two groups, the average latency and the tail latency, where tail latency is the highest possible latency of all users of the system. 

#### Throughput

Throughput is how many units of work the system can process in a unit of time. The higher the throughput of a system, the better. 

#### Benchmarking 

To measure the throughput of a system or server, the process is as generally follows: 
- Build a bunch of clients
- Have each send requests to the server
- Measure the latency of each request

We have two types of clinets, open and closed.

##### Open Client

An open client generates requests at a fixed rate. 

##### Closed Clinet

A closed clinet generates a new request each time the previous request finishes

A latency/throughput graph looks generally constant until the 'knee' of the graph is reached, or the point where the requests come in faster than they can be handled, at which point is tends towards infinity. The 'knee' of the graph is the point where the system acheives its maximum throughput. 

### Bottleneck Law

The bottleneck law states that the throughut of a system is equal to the minimum throughput of all modules. This module is referred to as the 'bottleneck' and the law states that this is the section that should be optimized. The speedup of a system after a performance optimization can be found with Amdahl's Law.

### Other Methods

#### Fast Paths 

Fast paths improve optimization by making the most commonly accessed features quicker to access, reducing average latency and increasing throughput, though in the worst case, creating or implementing a fast path can increase tail latency. 

#### Batching

Batching is the mechanism of grouping together a bunch of requests and processing them all at the same time to amortize overhead. This will increase throughput and reduce average latency, though will increase the tail latency due to **dallying**. 

##### Dallying

Dallying is the process of delaying the processing of a request to wait for more requests, to form a batch. 

#### Parallelism

Having seperate workers to process requests independently will increase the throughput, though the latency of the system will remain unchanged. 

Splitting the processing among multiple works will increase the throughput and reduce the latency of the system. 

--- 

## Caching

Caches are extremely fast, though relatively small storage that keeps a subset from a large, slow storage to reduce access time. The cache usually sits between the program and the storage. 

### Cache Hit and Miss

A cache hit is when you request an item from the cache and the item is inside the cache.

A cache miss is when you request an item from the cache and the item is not inside the cache, rather is in the slow storage.

Calculating Average Memory Latency: (hit rate)(hit latency) * (miss rate)(miss latency)

### Working Sets

A working set at some time t is the set of distinct items that the program has accessed in the past/most recent k operations. It is denoted W(t, k).

#### Temporal Locality

A program is likely to access items that it has recently accessed

#### Spacial Locality 

A program is likely to access items that are 'near' items that it has recently accessed, in regards to address space. 

### Caching Policies

A caching policy is how the cache decided to evict an item when the cache is full and needs room for a new element. There are two main methods, FIFO and LRU

#### FIFO

First in First Out(FIFO) says that when the cache is full, evict the item that has been inside the cache for the longest, or the first added element. This is an easy and efficient caching policy to implement into systems. 

#### LRU

Least Recently Used(LRU) says that when an item needs to be evicted, evict the item that was used least recently, or longest ago. This method is much more difficult and inefficient to implement.

### Time to Live

Time to live is another caching policy which gives each item a maximum time to live. Once that time hits 0, the cache will evict the item until it is stored into the cache again. 

--- 
