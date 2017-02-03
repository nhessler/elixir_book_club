# Elixir in Acton - Chapter 5 - Concurrency primitives

## BEAM (virtual machine)
- Runs in a single OS process

#### BEAM schedulers
- Each is an OS thread
- Manage concurrent BEAM processes
  + By default, as many schedulers as there are CPU cores available

#### BEAM processes
- Lightweight concurrent threads of execution
- Lighter and cheaper than OS process
  + A couple of microseconds to create
  + Initial memory footprint: 1-2 KB
- Everything runs in a process
- Completely isolated from one another sharing no memory
- Store an immutable structure and keep it for a long time
- Receive messages from other processes

---

## Concurrency

#### Concurrency vs Parallelizm
- Concurrency means multiple things have independent execution contexts
- Concurrent things can be executed in series, e.g., when only one CPU core available

#### Nothing-shared concurrency
- When we send a message from one process to another, the BEAM will deep-copy it.
  + Exception: binaries larger than 64 bytes
    * Maintained on a special shared binary heap
    * Useful when sending a message to many processes.
- Avoid sending large complex data.

#### Benefit of running tasks in different processes
- Parallelize the work as much as possible
- Improve the server's reliability and fault tolerance

#### Benefit of the nothing-shared concurrency
- Simplify each process
- No need for complex synchronization
- GC takes place at the process level, which will not interrupt the rest of the system

---

## Concurrent parallel executions
- To run something concurrently, we need to create a separate process using `spawn/1`
- The order of execution is not guaranteed

#### "fire and forget" concurrent execution

```elixir
# Define a lambda that simulates a long-running database query.
run_query = fn(query_def) ->
  :timer.sleep(2000)  
  "#{query_def} result"
end

# Running once in the current process. It takes 2 seconds.
run_query.("masatoshi")

# Running multiple times in the current process. It takes 10 seconds.
1..5 |> Enum.map(&run_query.("query #{&1}"))
["query 1 result", "query 2 result", "query 3 result", "query 4 result", "query 5 result"]
```

```elixir
# Define a lambda that spawns a process for a specified query.
async_query = fn(query_def) ->
  spawn(fn ->
    IO.puts(run_query.(query_def))
  end)
end

# Running once in a separate process for each query. It takes 2 seconds.
1..5 |> Enum.each(&async_query.("query #{&1}"))
```

#### Message passing
- The content of a message can be any Elixir term.
- A message is deep-copied when it is being sent.

```elixir
current_process = self

# Send a message from the current state.
send(current_process, "Hello, how are you?")

# Wait for a new messsage to arrive.
# Works similarly to the case expression but does not raise an error when the message
# did not match any pattern.
receive do
  message -> IO.inspect(message)

  # In case that no message was received in a given time frame (milliseconds)
  after 5000 -> IO.puts "Message not received"
end
```

```elixir
current_process = self

# Spawn a process for sending a message back to the current process.
spawn(fn ->
  send(current_process, {:message, "Hello world"})
end)

# Receive the message.
receive do
  {:message, message} -> IO.puts(message)
end
```

```elixir
# Define a lambda that simulates a long-running database query.
run_query = fn(query_def) ->
  :timer.sleep(2000)
  "#{query_def} result"
end

# Define a lambda that performs a database query and sends the result back to the caller's process.
async_query = fn(query_def) ->
  caller  = self
  message = {:query_result, run_query.(query_def)}
  spawn(fn ->
    send(caller, message)
  end)
end

# Define a lambda that receives messages.
get_result = fn ->
  receive do
    {:query_result, result} -> result
  end
end   
```

## Stateful server processes
- Resemble objects
- Endless loops that should be implemented with the tail recursion
- Wait for a message in each step of loop
- A standard abstraction `gen_server` (generic server) will simplify our developing stateful server processes.
- Use cases:
  + Database connection
  + TCP connection

---

## Runtime considerations
### 1. Although multiple processes may run in parallel, a single process is always sequential
- First, try to make a server handle a message quickly enough
- As a last resort, split the server into multiple processes

### 2. The mailbox size is limited by available memory
- Do not bite off more than you can chew
- Having a large amount of messages waiting in the mailbox can slow down the system
- Ensure that all the messages are handled
- Introduce a clause that match and deal with all the unexpected types of messages
