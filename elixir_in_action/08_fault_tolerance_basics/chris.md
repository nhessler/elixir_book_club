# Fault-tolerance basics

_"Let it crash, let it crash..."_

Fault tolerance is a first-class concept in BEAM.
The aim of fault tolerance is to acknowledge the existence of failures,
minimize their impact, and ultimately recover without human intervention.
Instead of trying to reduce the number of errors, OTP applications minimize
their effects and recover from them automatically.

## TLDR

- Three types of runtime errors: `throws`, `errors`, and `exits`
- When runtime errors occur, execution moves up the stack to a
  corresponding `try` block
- Process termination can be detected in another process
  - _Links_ are bidirectional
  - _Monitors_ are unidirectional
- When a process terminates abnormally, all linked processes terminate
- Supervisors can be used to start, supervise, and restart crashed processes

## The Details

### 8.1 Runtime errors

One of the most common runtime errors is a failed pattern match.

### 8.1.1 Error types

BEAM has three types of runtime errors: `errors`, `exits`, `throws`

You can raise an error by using the `raise/1` macro.

> **CONVENTION**: If your function explicitly raises an error,
  you should append the `!` character to its name.

### 8.1.2 Handling errors

The main tool for handling errors is the `try` construct.
If a runtime error isn't handled, the corresponding process will terminate.

Two things are specified in a `catch` block:
- `error_type` will contain an atom: `:error`, `:exit`, or `:throw`
- `error_value` will contain error-specific information

With `try`, a return value is the result of the last exected statement:
- either from the `do` block or
- if an error was raised, from the `catch` block

The `after` block is always executed, therefore it is useful for
cleaning up resources. NOTE: the `after` clause doesn't affect
the result of the entire `try` construct.

Custom errors can be defined with the `defexception` macro.

> **CONVENTION**: A common idiom is to _let the process crash_ and then
  do something about it, such as restart the process.

### 8.2 Errors in concurrent systems

### 8.2.1 Linking processes

`links` are the basic primitive for detecting a process crash.
- `links` connect exactly two processes
- `links` are always bidirectional
- `Process.link/1` connects the current process with another process
- `spawn_link/1` spawns a process and links it to the current one

An exit signal contains the pid of the crashed process and the exit reason.

### 8.2.2 Trapping exits

- Links break process isolation and propagate errors over process boundaries
- `Process.flag(:trap_exit, true)` makes the current process trap exit signals
- If a process traps exits, it isn't taken down when a linked process crashes

### 8.2.3 Monitors

Unlike links, monitors are unidirectional.

- `Process.monitor(target_pid)` is used to monitor a process
- `Process.demonitor(monitor_ref)` stops the monitor

If the monitored process dies, your process receives a message:

```elixir
{:DOWN, monitor_ref, :process, from_pid, exit_reason}
```

### 8.3 Supervisors

Supervisors are the primary tool of error recovery in concurrent systems.

### 8.3.1 Supervisor behavior

The generic `:supervisor` module works as follows:

1. The behavior starts and runs the supervisor process
2. The supervisor process traps exist
3. From the supervisor process, child processes are started and linked
   to the supervisor process
4. If a crash happens, the supervisor process receives an exit signal
   and performs the correct action (i.e. restarting the child process)
5. If a supervisor is terminated, child processes are terminated immediately

### 8.3.2 Defining a supervisor

- Workers are defined using `Supervisor.Spec.worker/2`
- `start_link` is used to start a process and link it to the caller
- The `one_for_one` strategy means "if a child crashes, start another"

### 8.3.3 Starting a supervisor

- `Process.whereis/1` returns pid
- `:kill` is treated in a special way, it ensures that the target process
  is unconditionally taken down, even if the process is trapping exits
- Aliases allow for process discovery

### 8.3.4 Linking all processes

When the supervisor starts the to-do cache, you get a completely new process
heirarchy, so this could introduce dangling processes.

To ensure proper cleanup, you need links.

### 8.3.5 Restart frequency

The supervisor relies on the `maximum restart frequency`, which defines how
many restarts are allowed in a given time period.

If you exceed the restart frequency, the supervisor will give up.
