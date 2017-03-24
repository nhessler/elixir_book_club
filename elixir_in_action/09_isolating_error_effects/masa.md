# Elixir in Action - Chapter 9 - Isolating error effects

- Fine-grained supervision (isolating errors and minimizing their negative effects)
- Supervisors allows us to localize the impact of an error.
- Each process should be somewhere in a supervision tree so that we can terminate it by terminating the supervisor.
- When a process crashes, its state is lost. We can either store state outside the process or start with a clean slate.
- In general, errors should be handled through a supervision hierarchy, not `try`.

---

## Linked processes

#### Default linked processes
- Regardless of which process crashes, the exit signal will be propagated to its linked processes.

#### Supervised linked processes
- The supervisor traps exit signals, thus preventing further propagation.

---

## Supervisor
- All processes that started directly from a supervisor should be OTP-compliant processes.
- Has the ability to stop the entire system without leaving dangling processes.

### Child processes
- Started synchronously in order specified.
- If restarts happens too often, the supervisor gives up at some point and terminates all of its children.
- A failed restart of one child will terminate the entire system.

### Starting child processes dynamically
- Sometimes we do not know how many children we need or what argument should be passed when starting child processes.
- Use a `simple_one_for_one` supervisor.
- Call `Supervisor.start_child/2` function in order to start a child process dynamically.

### Shutting down processes

#### Default termination
- We can terminate all the lined processes when the top-level supervisor process is terminated.
- When a supervisor is terminated:
  + it terminates all of its immediate children and all other processes directly or indirectly linked to those children.
  + restarts a terminated process regardless of the exit reason (even when it is `:normal`)

#### Doing final cleanup using `terminate/2` callback
  + 1. Set up an exit trap from `init/1` callback
  + 2. Invoke `terminate/2` callback

#### Controlling how long the supervisor will allow its child to terminate
  + An integer indicating a time in milliseconds
  + `:infinity` - wait indefinitely for the child to terminate
  + `:brutal_kill` - immediately terminate the child in a forceful way

#### Avoiding process restarting (temporary workers)
  + Prevent a worker from restarting on termination
  + HTTP request, TCP connection, etc
  + `worker(module, args, restart: :temporary)`

#### Restarting only on abnormal termination (transient workers)
  + In case we want to terminate a process normally to save memory.
  + `worker(module, args, restart: :transient)`

---

## Process registry

- A simple `gen_server` that maintains the alias-to-pid mapping.
- In real life, we want to uses:
  + [Registry](https://hexdocs.pm/elixir/Registry.html) [released in Elixir 1.4](http://elixir-lang.org/blog/2017/01/05/elixir-v1-4-0-released/)
  + gproc (a third-party library)

#### Why we need a process registry?
- If a process is started from a supervisor, we don't have access to the pid of the worker process.
- We can't keep a process's pid for a long time, because that process might be restarted and its successor will have a different pid.
- Elixir aliases can only be atoms; thus we need to maintain a key-value map, where the keys are aliases and the values are pids.

```
1. The worker process registers itself during initialization.
2. The client process will query the registry for the pid of the desired worker.
3. The client can then issue a request to the server process.

Performed internally by GenServer through via-tuples
#===================================================#
#                                                   #
#                    2. PROCESS DISCOVERY           #
#                    whereis_name(complex_alias)    #
#  ProcessRegistry <--------------------------------#---------
#    ^                                              #          \
#    |                                              #           \
#    | 1. PROCESS REGISTRATION                      #            \
#    | register_name(complex_alias, pid)            #          Client
#===================================================#            /
     |                                                          /
     |                                                         /
  Process <---------------------------------------------------
                     3. REQUEST
```

#### The `gen_server` behaviour compatible interface
- `register_name(complex_alias, pid)` returns:
  + `:yes` - if a successful registration
  + `:no` - if a process under the given alias is already registered

#### Via tuples
- Must be in the form `{:via, module, alias}`.
- We can send via-tuples to `GenServer` function such as `start_link`, `call`, and `cast`.
- `GenServer` will:
   + use functions from the given module to register a process
   + discover it
   + send messages to it

---

## Accessing a specific process
- To issue a request to a specific process, we must either have its **pid** or know its **registered alias**.

#### a process accessed by its *pid*

```elixir
GenServer.start_link __MODULE__,
                     initial_state
```

#### a process accessed by its *registered alias*

```elixir
GenServer.start_link __MODULE__,
                     initial_state,
                     name: :database_server
```

#### a process accessed via a *process registry* using a complex alias

```elixir
GenServer.start_link __MODULE__,
                     initial_state,
                     name: {:via, Todo.ProcessRegistry, {:database_worker, 1}}
```

---

## OTP-compliant processes
- Use OTP-compliant processes whenever possible.

#### Characteristics
- comply with OTP design principles
- handle OTP-specific messages

#### Running OTP-compliant processes
- We can use the following to run OTP-compliant processes.
  + OTP behaviours: `gen_server`, `supervisor`, etc
  + Elixir abstractions: `Agent`, `Task`, etc

---

## Restart strategies

#### `one_for_all`
When a child crashes, the supervisor terminates all other children as well and then starts all children.

#### `rest_for_one`
When a child crashes, the supervisor terminates all processes from the child specification that are listed after the crashed child.
Then the supervisor starts new child processes in place of the terminated ones.

---

## Error handling
- As a general rule, if you know what to do with an error
  + handle it
- Otherwise, for anything **unexpected**
  + let the process crash
  + ensure proper error isolation and recovery via supervisors

### intentional programming aka Let it crash

- The code states the programmer's intention, rather than being cluttered with all sorts of defensive constructs
- Promotes a clean-slate recovery

### Explicit error handling
- For an error that can be dealt with in a meaningful way

#### Critical processes that should not crash
- A system's **error kernel**, etc
  + 1. Use defensive `try`/`catch` statements in each `handle_*` callback of a critical process
  + 2. Set up a proper supervision hierarchy to ensure termination of all dependent processes in case of an error-kernel process crash.

#### Expected errors
- If we can predict an error and have a way to deal with it, there is no reason to let the process crash.

```elixir
case File.read(...) do
  {:ok, contents} -> do_something_with(contents) # expected
  {:error, :enoent} -> nil                       # expected
  # If anything else happens, a pattern match will fail and the process will crash.
end
```

---

## Preserving the state
- State is not preserved when a process is restarted.
- As a rule, the state should be persisted after all transformations are completed.
- Persistent state can have a negative effect on restarts (e.g. invalid state)
- If we can afford to it is better to start clean and terminate all other dependent processes.
