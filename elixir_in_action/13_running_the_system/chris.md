# Running the system

## TLDR

- To start an OTP system, code needs to be compiled,
  the BEAM needs to be started, and the OTP applications
  need to be started
- OTP releases are a standalone system
- When running in prod, compile using the prod MIX_ENV

## The Details

### 13.1 Running a system with the mix tool

Running the system is basically:
- Compiling all the modules
- Starting the BEAM instance
- Starting all the OTP applications

When you run `iex -S mix`, it does all of that.

### 13.1.1 Bypassing the shell

- `mix run --no-halt` runs the system without the shell
- `elixir -S mix run --no-halt` allows you to provide additional arguments
for the Erlang VM

### 13.1.2 Running as a background process

- `--detached` flag detaches the terminal, so there's no console output
- `epmd -names` displays running BEAM nodes

Remote Shell
- `--remsh` allows you to connect to a remote shell

Stopping a Node
- `:init.stop` shuts down all running applications and terminates the BEAM instance

### 13.1.3 Running scripts

An `escript` is a single binary file that embeds all .beam files and startup code.

### 13.1.4 Compiling for production

- `Mix.env` only has meaning during compilation
- When running in production, use the `:prod` environment: `MIX_ENV=prod elixir -S mix run --no-halt`

Protocol consolidation makes protocol dispatch as efficient as possible.

- `mix compile.protocols` generates efficient dispatching code
- Before load or performance testing, consolidate protocols first

### 13.2 OTP releases

_OTP releases_ are standalone, compiled, runnable systems consisting of the minimum set of OTP applications needed by the system.

_Release handling_ is Erlang's way of rolling out system upgrades.

[exrm](https://github.com/bitwalker/exrm) gives you a compile-time `mix release` task. 

Once you have this dependency:
- `mix deps.get`
- `MIX_ENV=prod mix compile --no-debug-info`
- `MIX_ENV=prod mix release`

This produces a fully standalone release.

> **WARNING**: Unlike a remote shell, when you attach to the shell of a BEAM
  node, commands run in the context of the running node.  If you hit CTRL+C
  twice while attached to a node, you will terminate the node.

### 13.2.3 Release contents

A standalone release consists of:
- Erlang runtime binaries
- Compiled OTP applications needed to run the system
- `vm.args` containing arguments that will be passed to the VM
- Boot script describing which OTP applications need to be started
- `sys.config` file containing environment variables for OTP applications
- Helper shell script to start, stop, and interact with the system

> **WARNING**: If your system cannot tolerate any downtime, live upgrades
  can upgrade the system on the fly.  However, server state or changes
  to the supervision tree can complicate things, so use live upgrades
  with caution.

### 13.3 Analyzing system behavior

[Erlang in Anger](http://www.erlang-in-anger.com)

### 13.3.1 Debugging

- Pipe (`|>`) into `IO.inspect` without affecting the behavior
- `IEx.pry/1` is another useful way to stop the system and inspect
  the state at a given point in execution
- `etop` is like the UNIX `top` command for BEAM
- Tracing can be done with `:sys.trace/2`
