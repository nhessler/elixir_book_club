# Concurrency primitives

In BEAM, processes are the fundamental unit of concurrency.

## TLDR

- A BEAM process is an isolated lightweight concurrent unit of execution 
  that shares no memory
- Processes communicate using asynchronous messages. Synchronous messaging
  must be built manually
- A server process is a process that runs for a long time (possibly forever)
- Server processes can maintain their own private state

## The Details

### 5.1 Principles

Challenges of highly available systems:

- **Fault tolerance**: minimize, isolate, and recover from errors
- **Scalability**: handle changes in load without changing or redeploying code
- **Distribution**: run a system on multiple machines 

In BEAM, a process is a concurrent thread of execution.

- Unlike OS processes/threads, BEAM processes are lightweight
  (1-2KB memory footprint)
- By default, BEAM uses 1 scheduler per CPU core
- BEAM processes are completely isolated
  - Share no memory
  - Crash of one process does not affect other processes
  - BEAM provides a mechanism to detect a process crash and handle it
    (i.e., restart)

### 5.2 Working with processes

### 5.2.1 Creating processes

Processes are created with the `spawn/1` function.

```elixir
spawn(fn ->
  :timer.sleep(1000)
  IO.puts("wow. such process.")
end)
```

One way to pass data to a process is to wrap the lambda in a closure:

```elixir
async_process = fn(data) ->
  spawn(fn -> IO.puts(data) end)
end

async_process.("orly")
async_process.("yes")
```

### 5.2.2 Message passing

The caller process doesn't get the result of the spawned processes,
so how do processes communicate?

- Processes can communicate via asynchronous messages
- Content of a message is an Elixir term
  (anything that can be stored in a variable)
- Process mailbox is a FIFO queue limited only by the available memory

To send a message to a process:

- you need to know the pid
- use the `Kernel.send/2` function

```elixir
send(pid, message)
```

To receive a message, use the `receive` construct.

```elixir
receive do
  pattern_1 -> do_something
  pattern_2 -> do_something_else
  after 1000 -> IO.puts("Timeout")
end
```

- `receive` waits indefinitely for a new message to arrive
- The `after` clause sets a timeout in milliseconds

> **NOTE**: `receive` does not throw an error if the pattern does not match.
  Instead, it leaves the message in the process mailbox and the next message
  is processed. As a result, if messages are left in the process mailbox,
  you can run out of memory and performance will degrade.
  It's good to use a catch-all pattern to consume unwanted messages.

- Synchronous sending must be implemented on top of asynchronous messages
  (asynchronous by default is great)

### 5.3 Stateful server processes

Server processes are long-running processes that respond to messages.
Think of this like the Actor pattern.

> **CONVENTION**: When implementing a server process, it usually makes sense
  to put all of the code in a single module.

- _Interface functions_ are public functions executed in the caller process
- _Implementation functions_ are usually private and run in the server process 

> **CONVENTION**: `gen_server` is a standard abstraction for the pattern
  of creating stateful server processes, but it's in the next chapter ;)

It's important to remember that each server process is sequential internally.

- If you issue 1,000 asynchronous calls to a single server process,
  they will be handled in FIFO
- A pool of processes can be used to handle multiple requests concurrently

### 5.3.2 Keeping a process state

Server processes allow keeping state for as long as the process is running.
This usage is almost like an object reference in an object oriented language.

### 5.3.3 Mutable state

By sending messages to a process, a caller can affect its state and the
outcome of subsequent requests handled in that server.

### 5.3.4 Complex states

When state gets complex, it's worth extracting the state manipulation
to a separate module (separation of concerns).

Concurrent vs. functional approach:

- A process that keeps mutable state is like a mutable data structure
- This feature could be abused, so if you find yourself relying on this,
  take a step back and think about whether the problem could be modeled
  with pure functional abstractions

### 5.3.5 Registered processes

A `pid` resembles a reference or pointer in the OO world.

- If you know there will always be only one instance of a server,
  you can `register` it under a `local alias`
- The alias is `local` because it only has meaning for the currently
  running BEAM instance

`Process.register(pid, :alias)` can be used to register a process.

- The process alias must be an atom
- A single process can only have one alias
- Two processes cannot have the same alias

Registered aliases provide a way of communicating with a process without
knowing its pid.

### 5.4 Runtime considerations

### 5.4.1 A process is a bottleneck

Although multiple processes can run in parallel, a single process is always
sequential.

Back to the concurrency vs. parallelism idea, if many processes all send
messages to a single process, that process becomes the bottleneck.

### 5.4.2 Unlimited* process mailboxes

- Process mailboxes are theoretically unlimited, but practically limited by
  available memory
- If tons of messages are sent to a single slow process, it could cause the
  system to crash by consuming all of the available memory
- You should use a `match all` receive clause to deal with unexpected messages
  blowing up the process mailbox

### 5.4.3 Shared nothing concurrency

- Sending a message to another process results in a deep copy
- A special case where deep copying doesn't happen involves binaries
  that are larger than 64 bytes.  They're maintained on a special
  shared binary heap
- Since processes don't share memory, garbage collection happens on a process
  level.  When more memory is needed, GC takes place and happens in the
  process execution window. There is no stop the world GC! :tada:

### 5.4.4 Scheduler inner workings

- Usually 1 scheduler per CPU core with significantly more processes
- Internally, each scheduler has a run queue, which is a list of BEAM
  processes that it's responsible for
- BEAM does implicit yielding for `receive`, `:timer.sleep/1`, and I/O
- I/O operations are internally executed on separate async threads
