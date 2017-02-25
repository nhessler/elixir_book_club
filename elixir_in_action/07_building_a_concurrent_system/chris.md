# Building a concurrent system

_"Now you're thinking with ~~portals~~ processes."_

This chapter is about learning to think in terms of processes
and analyzing the system for bottlenecks.

## TLDR

- `mix` is good for managing systems with multiple modules 
- Running tasks in separate server processes can promote scalability
  and fault-tolerance of the overall system
- Processes are internally sequential
- Carefully consider calls vs. casts for the impact to the system

## The Details

### 7.1 Working with the mix project

- `mix` is used to create, build, and run projects, as well as manage 
  dependencies, run tests, and run custom project tasks
- `iex -S mix` starts a shell with project modules loaded
  (kind of like `sbt console` for Scala people)

**CONVENTION**:

- Modules should be placed under a common top-level alias
  (such as `Todo.Server` and `Todo.Database`)
- One file should contain one module
- Filenames should be snake case versions of the module name
  (`TodoServer` should be in `todo_server.ex`)
- Folder structure should correspond to multipart file names
  (`Todo.Server` should be in `lib/todo/server.ex`)

### 7.2.1 Implementing a cache

The `__MODULE__` construct is replaced with the name of the current module:

```elixir
GenServer.start(__MODULE__), nil)
```

### 7.2.2 Analyzing process dependencies 

- In the Elixir/Erlang world, HTTP servers typically use a separate
  process for each request
- Since processes only handle one request at a time, the internal
  state is consistent, which makes race conditions in a single
  process impossible
- If you need to synchronize a block of code, it's best to run that
  code in a dedicated process

### 7.3 Persisting data 

- A long-running `init/1` function will cause the creator process to block
- A hack is to use `init/1` to send itself an internal message that
  initializes the process state in a `handle_info` callback
- This works as long as the process isn't registered under a local alias

### 7.3.3 Analyzing the system

- If requests to the DB come in faster than they are handled, the
  process mailbox will grow and consume increasing amounts of memory
- When a request times out, it isn't removed from the receiver's mailbox

### 7.3.4 Addressing the process bottleneck

**Reasons to run code in a dedicated server process**:

- Code must manage long-living state 
- Code handles a resource that should be reused,
  such as a network or DB connection
- A critical section of the code must be synchronized

**Handling requests concurrently**:

- For synchronous calls, the response can be returned from a spawned
  worker process, but the problem is that concurrency is unbound
- Pooling can be achieved by computing a hash to fall in the range
  of `0..N-1` and forwarding the request to a worker in a pool of size `N`
- [poolboy](https://github.com/devinus/poolboy) is a popular pool library

### 7.4 Reasoning with processes 

_"Call me, maybe?"_

**Casts**:

- Casts promote system responsiveness (because a caller isn't blocked)
  at the cost of reduced consistency
- Using casts may allow clients to overload the server and allow 
  requests to pile up in the message box and consume memory

**Calls**:

- Calls promote consistency (because a caller gets a response), but
  reduces system responsiveness
- Calls can be used to apply backpressure to client processes, preventing
  them from generating too much work
