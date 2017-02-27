# Elixir in Action - Chapter 7 - Building a concurrent system

## mix tool
- create a project
- build a project
- run a project
- manage the project's dependencies
- write custom project-based tasks

#### Create a project

```
$ mix new todo
```

#### Compile the project

```
$ mix compile
```

#### Run a test

```
$ mix test
```

#### Compile and run in the Elixir shell
- Compile the project
- Start the shell
- Make available all the compiled modules in load paths

```
$ iex -S mix
```

#### The `lib` folder
- Place `.ex` files under the `lib` folder
- Automatically included in the next build

#### Conventions
- Place our modules under a common top-level alias
  + e.g., `Todo.List`, `Todo.Server` etc
- In general, one fie should contain one module
  + exceptions:
    * small helper modules
    * implementing protocols for the module
- A filename should be a snake case of the main module name that it implements
- The folder structure should correspond to multipart module names
  + e.g., `Todo.Server` -> `lib/todo/server.ex`

---

## Managing multiple server processes

- Create another entity with the key-value structure that fetches an existing server or create one.

#### List the pids of all the currently running processes
- Use `:erlang.processes/0`
- Erlang and Elixir themselves use some process internally.

```elixir
iex> :erlang.processes
# [#PID<0.0.0>, #PID<0.1.0>, #PID<0.4.0>, #PID<0.30.0>, #PID<0.31.0>,
#  #PID<0.33.0>, #PID<0.34.0>, #PID<0.35.0>, #PID<0.36.0>, #PID<0.38.0>,
#  #PID<0.39.0>, #PID<0.40.0>, #PID<0.41.0>, #PID<0.42.0>, #PID<0.43.0>,
#  #PID<0.44.0>, #PID<0.45.0>, #PID<0.46.0>, #PID<0.47.0>, #PID<0.48.0>,
#  #PID<0.49.0>, #PID<0.50.0>, #PID<0.51.0>, #PID<0.52.0>, #PID<0.53.0>,
#  #PID<0.59.0>, #PID<0.60.0>, #PID<0.61.0>, #PID<0.62.0>, #PID<0.63.0>,
#  #PID<0.65.0>, #PID<0.66.0>, #PID<0.67.0>, #PID<0.68.0>, #PID<0.83.0>,
#  #PID<0.84.0>, #PID<0.85.0>, #PID<0.86.0>, #PID<0.87.0>, #PID<0.88.0>,
#  #PID<0.97.0>, #PID<0.98.0>, #PID<0.99.0>, #PID<0.100.0>, #PID<0.101.0>,
#  #PID<0.102.0>, #PID<0.103.0>, #PID<0.104.0>, #PID<0.106.0>]
```

```elixir
iex> length(:erlang.processes)
# 49
```

#### Creating 100,000 todo servers

```elixir
iex> {:ok, cache} = Todo.Cache.start
# {:ok, #PID<0.109.0>}
iex> cache
# #PID<0.109.0>
iex> length(:erlang.processes)
# 50

iex> 1..100_000 |> Enum.each(fn(i) -> Todo.Cache.server_process(cache, "todo_list #{i}")  end)
# :ok
iex> length(:erlang.processes)
# 100050
```

---

## Encoding and persisting

- `:erlang.term_to_binary/1` and `:erlang.binary_to_term/1` for encoding/decoding between binary and term.
- `File.write!/2` and `File.read/1`

```elixir
iex> b = :erlang.term_to_binary("hello")
# <<131, 109, 0, 0, 0, 5, 104, 101, 108, 108, 111>>
iex> :erlang.binary_to_term(b)
# "hello"
```
---

## HTTP servers in Erlang and Elixir
- HTTP servers typically use separate processes for each request.
- Regardless of how many concurrent requests are coming in, a single process can handle only one request at a time, which can be a bottleneck.
- Each list is managed by a single process, so a single list cannot be modified by two simultaneous clients.

#### Evaluate the request-handling rate
- Evaluate how many requests per second the Todo.Cache server can handle.
  + If a request takes 0.000001 seconds, we can serve 1M requests/sec.
  + If a request takes 0.1      seconds, we can serve 10 requests/sec.
- The request-handling rate must at least be equal to the incoming rate.

```elixir
# Multiple clients issue requests to the single `Todo.Cache` process.

         issue a request for
          a server pid         ________________  handles one at a time
client 1 -------------------> |                | (creating / fetching a Todo.Server process)
client 2 -------------------> |   Todo.Cache   | ---------> Todo.Server
client N -------------------> |________________|            
```

```elixir
            uses                                            
client 1 -------------------------------------------------> Todo.Server 1

            uses                                            
client 2 -------------------------------------------------> Todo.Server 2

            uses                                            
client N -------------------------------------------------> Todo.Server 3
```

---

## Reasons for running a piece of code in a dedicated server process instead of client process
- The code must outlive a long-living state
- The code handles a kind of resource that can and should be reused
  + e.g., TCP connection, database connection, file handle, pipe to an OS process, etc
- A critical section of the code must be synchronized; only one process may run this code in any moment
- If computations can safely run in parallel, consider running them in separate processes.
- If an operation must be synchronized, consider running it in a single process.

---

## Potential bottleneck of a singleton server handling multiple call requests
- A Todo.Server issue a synchronous request, and wait until the database returns the response. It cannot handle new messages while waiting for a response.
- Even if a call request times out, that message stays in the receiver's mailbox waiting to be processed.
- Can cause the entire system to be useless under a heavy load.

```elixir
               persist and get      ________________  handles one at a time
Todo.Server 1 -------------------> |                | (reading / writing data)
Todo.Server 2 -------------------> | Todo.Database  | ---------> database operations
Todo.Server N -------------------> |________________|
```

---

## Handling requests concurrently
- Run each database operation in a spawned one-off process
- Multiple processes run concurrently but a single process handles requests sequentially
- Limit concurrent operations with pooling
- Per-key synchronization
  + Make requests for the same item end up in the same worker
  + The worker becomes the synchronization point for the single item

```elixir
                                   handles requests   
                                   sequentially
               persist and get      ________________  hash       handle database operations concurrently
Todo.Server 1 -------------------> |                | ---------> worker a ---------> database operations
Todo.Server 2 -------------------> | Todo.Database  | ---------> worker b ---------> database operations
Todo.Server N -------------------> |________________| ---------> worker c ---------> database operations
```

---

## Communication between service processes

#### call requests
- promotes consistency.
- used when the client needs a response.
- used when the client wants to ensure that the request is successfully handled.
- The client becomes synchronized with the server
- The client can never produce more work than the server can handle.

#### cast requests (aka fire-and-forget requests)
- promote system responsiveness by not blocking the client process.
- The clients may overload the server
- requests may pile up in the message box and consume memory.
