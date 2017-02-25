# Generic server processes

`gen_server` is a helper that simplifies implementation of server processes

## TLDR

- A _generic server process_ is an abstraction that implements tasks common
  to any type of server process
- A _generic server process_ can be implemented as a behavior
- The behaviour invokes callback functions when the specific implementation
  needs to make a decision
- `gen_server` is an OTP behavior that implements a generic server process
- A callback module for `gen_server` must implement various functions:
  - `init/1`
  - `handle_cast/2`
  - `handle_call/3`
  - `handle_info/2`
- `GenServer` module can be used to interact with `gen_server` processes
- Two types of requests can be issued to a server process:
  - _Cast_ is an asynchronous fire-and-forget request
  - _Call_ is a synchronous send-and-respond request

## The Details

### 6.1 Building a generic server process

All code that implements a server process needs to:
- Spawn a separate process
- Run an infinite loop in the process
- Maintain the process state
- React to messages
- Send response back to the caller

### 6.1.1 Plugging in with modules

- Make generic code accept a plug-in callback module
- Maintain the module atom as part of the process state
- Invoke callback module functions when needed

### 6.1.4 Supporting asynchronous requests

- _call_ for synchronous requests
- _cast_ for asynchronous requests
- Implementations can decide whether each concrete request should be
  implemented as a call or as a cast

### 6.2 Using gen_server

Some features of `gen_server`:
- Support for calls and casts
- Customizable timeouts for call requests
- Propagation of server-process crashes to client processes
- Support for distributed systems

### 6.2.1 OTP behaviors

In Erlang terminology, a _behaviour_ is generic code that implements
a common pattern.

OTP ships with a few predefined behaviours:
- `gen_server` - implementation of a stateful server process
- `supervisor` - error handling and recovery in concurrent systems
- `application` - components and libraries
- `gen_event` - event-handling support
- `gen_fsm` - finite state machine in a stateful server process

### 6.2.2 Plugging into gen_server

The `use` macro injects a bunch of functions into the calling module

### 6.2.5 Other gen_server features

Local registration is an important feature because it supports patterns
of fault-tolerance and distributed systems.

### 6.2.6 Process life cycle

Erlang is an accidental implementation of the _Actor model_ originally
described by Carl Hewitt. There are some disagreements about whether
Erlang is a proper implementation of the Actor model.

### 6.2.7 OPT-compliant processes

- OTP-compliant processes adhere to OTP conventions, so they can be used
  in supervision trees
- Tasks make it simple to start a concurrent job and collect the result later
- Agents are a simpler alternative to `gen_server`-based processes and
  are appropriate if the single purpose of the process is to manage
  and expose state