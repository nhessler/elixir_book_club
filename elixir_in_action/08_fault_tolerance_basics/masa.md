# Elixir in Action - Chapter 8 - Fault tolerance basics
Basic error-handling techniques in a concurrent system.

---

## The aim of fault tolerance
- Failure detection
- Minimizing the impact of failures
- Resilience and self-healing

---

## Runtime errors
- Error types - `:error`, `:exit` or `:throw`
- Error value - a value that can be an arbitrary term
- If not handled. the corresponding process will terminate

### Common situations where an runtime error occurs
- A failed pattern match
- A call request's timeout
- An invalid arithmetic operation
- A nonexistent function being invoked
- Explicit error signaling

### Error types

#### errors
- Functions with `!` explicitly `raise` a runtime error.
  + e.g., `File.open!`

```elixir
raise "Something went wrong"
# ** (RuntimeError) Something went wrong
```

#### exits
- Deliberately terminate a process, using `exit/1` with reason

```elixir
spawn(fn ->
  exit("I am done")
  IO.puts("This is not invoked")
end)
```

#### throws
- `throw/1` allows non-local returns
- Like [`goto`](https://en.wikipedia.org/wiki/Goto) statement of other languages
- Should avoid using this technique

```elixir
throw(:thrown_value)
# ** (throw) :thrown_value
```

### Custom errors
- We can define custom errors using `defexception` macro.

---

## Handling errors with `try`

- We can intercept any types of errors (:error, :exit or :throw) using `try`.
- The tail-optimization is not available with `try` because what is called in a
`try` block is not the last thing a function does.
- In Elixir, it is more common to let the process crash and restart another one,
starting with a clean slate.

```
1. A runtime error occurs
2. The execution moves up the stack to the corresponding `try` block if any.
3. If the error is not handled, the process will crash.
```

```elixir
# type    - :error, :exit or :throw
# value   - a value thrown, an error raised, etc
# return  - the result of the last executed statement
try do
  # ...
catch type, value ->
  # ...
end

# With pattern matching
try do
  # ...
catch
  type_pattern_1, value_pattern_1 ->
    # ...
  type_pattern_2, value_pattern_2 ->
    # ...
end

# With the match-all clause
try do
  # ...
catch
  _type, _value ->
    # ...
end
```

```elixir
try_helper = fn(fun) ->
  try do
    fun.()
    IO.puts "No error"
  catch type, value ->
    IO.puts """
    Error
      #{inspect type}
      #{inspect value}
    """
  end
end
```

```elixir
try_helper.(fn ->
  %{} = %{name: "Masatoshi"}
end)
# No error
# :ok

# ---
# Raise an error.
# ---

try_helper.(fn ->
  raise("Something wend wrong")
end)
# Error
#   :error
#   %RuntimeError{message: "Something wend wrong"}
# :ok

# ---
# Exit a process.
# ---

spawn(fn ->
  try_helper.(fn ->
    exit("I am done")
    IO.puts("This is not invoked")
  end)
end)
# Error
#   :exit
#   "I am done"
# #PID<0.194.0>

# ---
# Throw a value.
# ---

try_helper.(fn -> throw(:thrown_value)end)
# Error
#   :throw
#   :thrown_value
# :ok
```

```elixir
try do
  raise "Something went wrong"
catch
  _type, _value -> IO.puts "Error caught"  # match-all clause
after  
  IO.puts "Cleanup code"
end
# Error caught
# Cleanup code
# :ok
```

---

## Detecting process termination

### Isolated processes

- A crash in one process won't affect the others unless we explicitly want it to.

```elixir
spawn(fn ->
  IO.puts("Process 1 started")

  # This process is not affected when Process 1 crashes.
  spawn(fn ->
    IO.puts("Process 2 started")
    :timer.sleep(1000)
    IO.puts("Process 2 finished")
  end)

  raise("Something went wrong")

  IO.puts("Process 1 finished")
end)
# Process 1 started
# Process 2 started
# #PID<0.109.0>
# 22:46:46.159 [error] Process #PID<0.109.0> raised an exception
# ** (RuntimeError) Something went wrong
#     (stdlib) erl_eval.erl:670: :erl_eval.do_apply/6
#     (stdlib) erl_eval.erl:122: :erl_eval.exprs/5
# Process 2 finished
```

### Linked processes (bidirectional)

- Communication channel for providing notifications about process terminations
- One link connects exactly *two processes* and always *bidirectional*
- One process can be linked to an arbitrary number of other processes
- `Process.link/1` - connects the current process with another process
- `spawn_link/1` - spawns a process and connects it with the current process

```
            Process
          /         \
    Process        Process
       |
    Process
```

- If two processes are linked, and one of them terminates, the other process receives
an exit signal; ultimately, entire tree of linked processes will be taken down.

```elixir
spawn(fn ->
  IO.puts("Process 1 started")

  # This process is terminated when Process 1 crashes.
  spawn_link(fn ->
    IO.puts("Process 2 started")
    :timer.sleep(1000)
    IO.puts("Process 2 finished")
  end)

  raise("Something went wrong")

  IO.puts("Process 1 finished")
end)
# Process 1 started
# #PID<0.137.0>
# 23:18:37.278 [error] Process #PID<0.137.0> raised an exception
# ** (RuntimeError) Something went wrong
#     (stdlib) erl_eval.erl:670: :erl_eval.do_apply/6
#     (stdlib) erl_eval.erl:122: :erl_eval.exprs/5
```

### Trapping exits
- By trapping exits, a process can detect the crash of linked processes without getting terminated itself.
- An exit signal is placed in the surviving process's message queue in the form of a standard message.
- `Process.flag(:trap_exit, true)` - makes the current process trap exit signals.

```elixir
spawn(fn ->
  IO.puts("Process 1 started")

  Process.flag(:trap_exit, true)

  spawn_link(fn ->
    IO.puts("Process 2 started")
    raise("Something went wrong")
    IO.puts("Process 2 finished")
  end)

  receive do
    # Instead of getting terminated, an exit signal is placed in the surviving process's message queue in the form of a standard message.
    msg -> IO.inspect(msg)
  end

  # Process 1 was not terminated.
  IO.puts("Process 1 finished")
end)
# Process 1 started
# Process 2 started
# #PID<0.157.0>
# 23:47:06.195 [error] Process #PID<0.158.0> raised an exception
# ** (RuntimeError) Something went wrong
#     (stdlib) erl_eval.erl:670: :erl_eval.do_apply/6
#     (stdlib) erl_eval.erl:122: :erl_eval.exprs/5
# {:EXIT, #PID<0.158.0>,
#  {%RuntimeError{message: "Something went wrong"},
#   [{:erl_eval, :do_apply, 6, [file: 'erl_eval.erl', line: 670]},
#    {:erl_eval, :exprs, 5, [file: 'erl_eval.erl', line: 122]}]}}
# Process 1 finished
```

### Monitored links (unidirectional)
- If the monitored process terminates, the monitoring process receives a message in the format `{:DOWN, monitor_ref, from_pid, exit_reason}`.
- Only the process that created a monitor receives a monitor message.
- A single process can create multiple monitors.
- `Process.monitor(target_pid)` - start monitoring a process
- `Process.demonitor(monitor_ref)` - stop monitoring a process

```elixir
spawn(fn ->
  IO.puts("Observer process started")

  # Spawn a process that lasts one second.
  target_pid = spawn(fn ->
    IO.puts("Monitored process started")
    :timer.sleep(1000)
    IO.puts("Monitored process finished")
  end)

  # Monitor that process.
  monitor_ref = Process.monitor(target_pid)

  # Wait for a monitor message.
  receive do
    msg -> IO.inspect(msg)
  end

  IO.puts("Observer process won't crash when the monitored process terminates")
  IO.puts("Observer process finished")
end)
# Observer process started
# Monitored process started
# #PID<0.195.0>
# Monitored process finished
# {:DOWN, #Reference<0.0.4.806>, :process, #PID<0.196.0>, :normal}
# Observer process won't crash when the monitored process terminates
# Observer process finished
```

### Exits that propagate through `GenServer.call` requests

- `:gen_server` internally sets up a temporary monitor that targets the server process.
- If `:gen_server` receives a `:DOWN` message, it raises a corresponding exit signal in the client process.  

---

## Supervisors

- Each worker is supervised by a supervisor process.
- Whenever a worker terminate the supervisor starts another one in its place.
- The supervisor does nothing but supervise.

### OTP supervisor behaviour
- Implemented in `:supervisor` module.
- Elixir's `Supervisor` module provides a helper.

#### the supervisor behaviour's responsibilities
- The behaviour starts and runs the supervisor process.
- The supervisor process traps exits.
- From within the supervisor process, child processes are started and linked to the supervisor process.
- If a crash happens, the supervisor process receives an exit signal and performs corrective actions, such as restarting the crashed process.
- If a supervisor process s terminated, child processes are terminated immediately.   
```

#### the implementer's responsibilities
- In the callback module, we specify which children must be started and what todo when a child crashes.

#### Relationships between supervisor and worker processes

```
caller process              supervisor process               child processes
================================================================================
              start                      start and supervise
caller --------------------> :supervisor -------------------> child processes
                                  |                           (workers)
                                  |  get the spec
                                  v
                             CallbackModule.init/1                        
```

#### A link is required for a process to be supervised
- We must create a link if we want to have a process under a supervisor.
- Because links work in both directions, the termination of a supervisor means all of its children will be automatically take down.
- This allows us to properly terminate any part of the system without leaving behind dangling processes.

#### Terminating a process
- `Process.exit/2`

```elixir
Process.whereis(:todo_cache) |> Process.exit(:kill)
# Staring Elixir.Todo.Cache
# true
# Staring Elixir.Todo.Database
```

---

## Aliases allows process discovery

- Supervised processes can be restarted, being replaced by another process with a different pid.
- Registering an alias enables us to find a process regardless of possible process restart.

#### Getting the pid of a process that is registered under an alias
- `Process.whereis/1`

```elixir
defmodule Todo.Cache do
  use GenServer

  def start_link do
    GenServer.start_link(__MODULE__, nil, name: :todo_cache)
  end

  # ...
```

```elixir
Process.whereis(:todo_cache)
# #PID<0.122.0>
```

---

## Linking all the processes
- The primary way to achieve process consistency.
- We can ensure that the crash of one process takes down its dependencies as well.
- We can detect an error in any part of the system and recover from it without leaving behind dangling processes.
- Downside:
  + An error can have a wide impact.
  + An error in a single process will take down the entire structure.
