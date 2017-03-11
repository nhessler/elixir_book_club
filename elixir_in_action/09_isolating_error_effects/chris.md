# Isolating Error Effects

_"Supervise the supervisors!"_

This chapter is about isolating errors and minimizing their negative effects.

- By placing workers under a supervisor, you confine an error to a single
  worker
- By isolating errors under a single worker, other parts of the system can
  continue to run and provide service while the supervisor attempts to recover

## TLDR

- Supervision trees localize the impact of an error
- Each process should reside somewhere in a supervision tree.
  This prevents dangling processes and lets you terminate the entire system
- When a process crashes, its state is lost
- Handle unexpected errors through supervision tree (i.e. restarts)
- Explicitly handle errors if there is a meaningful way to deal with them

## The Details

### 9.1 Supervision trees

- To reduce error effects, start individual workers from the supervisor
- Remember that children are started synchronously in the order specified

### 9.1.2 Rich process discovery

A process registry is needed to maintain a key-value map from alias -> pid

- When a process is restarted, the successor should register itself under the
  same name so that it can be reached after multiple restarts
- Detect termination of a registered process and remove its entries from the
  registry
- Must implement and export `send/2` function to work with `GenServer`
  and `via` tuples
- To avoid name conflict, use `import Kernel, except: [send: 2]`

> **NOTE**: The implementation of the process registry is just for learning.
  [`gproc`](https://github.com/uwiger/gproc) is commonly used for this purpose.

### 9.1.3 Supervising database workers

- Each DB worker registers with the process registry under a unique alias
- Rely on the process registry to discover workers
- Create another supervisor that starts/supervises DB worker pool

If you keep too many children under the same supervisor, a failed restart of
a child process will terminate the entire system.

### 9.1.5 Starting the system

- Since supervisor processes are started synchronously in order specified,
  a child process must be specified after its dependencies
- In our case, the process registry must be started before the DB workers

### 9.1.6 Organizing the supervision tree

Supervision trees allow the system to attempt to recover locally from an
error, but if that doesn't work, move up the supervision tree to restart
a wider part of the system.

Plain processes started by `spawn_link` aren't OTP-compliant, so those
should not be started from a supervisor.

> **CONVENTION**: Use OTP-compliant processes wherever possible.
  Crashes in OTP-compliant processes are logged with more detail.

Graceful shutdown of a `gen_server` worker invokes `terminate/2` callback,
but only if the worker process is trapping exits.

If you want to do cleanup on termination, set up an exit trap in `init/1`.

A **shutdown strategy** lets you control how long the supervisor will wait
before forceful termination (think SIGTERM, then SIGKILL)
- Default is to wait at most 5 seconds
- Use `shutdown: <time in ms>` to specify custom time
- `:infinity` will wait forever

`:brutal_kill` is analogous to `Process.exit(pid, :kill)` which sends a
forceful exit signal to the process

By default, a supervisor restarts a terminated process regardless of the exit
reason:
- `worker(module, args, restart: :temporary)` creates a _temporary worker_
  while will not be restarted on termination
- worker(module, args, restart: :transient)` creates a _transient worker_
  that can be used for processes that may terminate normally as part of the
  standard workflow

### 9.2 Starting workers dynamically

`simple_one_for_one` supervisors allow the system to dynamically allocate
worker processes as long as:
- All the child processes are started by the same function
- No child is started upfront
- Children are started dynamically by calling `Supervisor.start_child/2`

> **CONVENTION**: It's idiomatic to use the `simple_one_for_one` strategy
  to indicate that you will start children dynamically

### 9.2.2 Restart strategies

- `one_for_all` - When a child crashes, the supervisor terminates all the
  other children as well and then starts all children.
- `rest_for_all` - When a child crashes, the supervisor terminates all 
  processes listed after the crashed process in the supervisor specification
  (if you recall, child processes should be listed in order of dependency)

### 9.3 "Let it crash"

_Let it crash_ means that you shouldn't worry about **unexpected** errors,
but you should still handle any error that you can deal with in a meaningful
way. 

- An important consequence of "let it crash" error handling is that the main
  worker code is free of defensive `try/catch` constructs
- Joe Armstrong coined this _intentional programming_ as it states the
  programmer's intention, rather than being cluttered with defensive code

_Let it crash_ promotes clean-slate recovery.

> **NOTE**: When a process is restarted, the old message queue is lost,
  which will cause some requests to fail (or be lost), but the new process
  will start fresh (without corrupted state)

### 9.3.1 Error kernel

The _error kernel_ consists of the processes that are critical for the
system to work and whose state can't be restored in a consistent way

If the code in the error kernel is too complex, consider splitting it
into two processes:
- One to hold state
- One to do the actual work

Errors that you should explicitly handle:
- Potential errors in critical processes that shouldn't crash
- Errors that can be dealt with in a meaningful way

Since the error kernel shouldn't crash, it makes sense to include
defensive `try/catch` statements in the `handle_*` callbacks for
critical processes.

Immutable data structures allow you to implement a fault-tolerant server:

```elixir
def handle_call(message, _, state) do
  try
    new_state =
      state
      |> transform_1
      |> transform_2
      ...
    {:reply, response, new_state}
  catch _, _ ->
    {:reply, {:error, reason}, state}
  end
```

Catching exceptions in the transformation allow the server to return
the original state, which is essentially a rollback of the transformations

### 9.3.2 Handling expected errors

The point of _let it crash_ is to leave recovery of unexpected errors
to supervisors.

### 9.3.3 Preserving the state

In some cases, you want process state to survive the crash.

The general approach is to save the state out of the process in another
process or to a file.

> **NOTE**: Be careful with persisting state, because it can have a negative
  effect on process restarts.  If the error is caused by invalid state
  and the state persists, the supervisor will never be able to restart the
  process successfully.

