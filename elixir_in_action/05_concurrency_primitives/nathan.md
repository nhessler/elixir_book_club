# 5 Concurrency Primitives

## 5.1

* Fault Tolerance, Scalability, Distribution!
* Beam processes are different than OS processes
* all of Beam runs in one OS process and an OS thread per CPU
* 268 Million processes as a virtual limit!

## 5.2

* you can pass data via closures from one process to another. it's passed via a deep copy.
* processes can only communicate via messages
* `self/1` returns the current process ID

## 5.3

* There is no special relation between modules and processes
* A server process can be thought of as a synchronization point
* keep state by passing parameters into your `loop` function
* async when you don't care about the result of another process, sync when you need a value back
* breaking the case statement of a receive into a multiclause function has some hidden issues
  * specifically, you now eat messages that don't match whereas they used to stay in the mailbox
  * i would avoid the multiclause function pass-off until you better understand receive
* register a process to avoid needing the pid
  * alias must be an atom
  * only one alias per process
  * each alias must be unique
  * module name is often used `__module__`

## 5.4

* while processes run concurrently, each process is itself sequential
* Parallelization is not a remedy for a poorly structured algorithm
* processes that get messages faster than they can handle them will grow in memory
* implement a match-all case in your receive block to keep the mailbox growing from unknown messages.
* each process gets about 2000 reductions during it's execution window
* I/O, sleep, and receive can end the window pre-emptively
* I/O is run on async threads (about 10 by default)
