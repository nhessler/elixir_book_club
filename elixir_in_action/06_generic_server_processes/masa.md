# Elixir in Action - Chapter 6 - Generic server processes

- Server processes are frequently used for building highly concurrent systems in Elixir and Erlang.

## Common tasks needed for implementing a server process
- Spawn a separate process
- Run an infinite loop in the process
- Maintain the process state
- React to messages
- Send a response back to the caller

---

## call vs cast
By the OTP convention:

- `call`: for synchronous requests
- `cast`: for asynchronous requests

---

## Generic server processes
- simplifies implementation of server processes
- performs tasks that are common to server processes
- leaves specific decisions to concrete implementations
  + initial state
  + how messages are handled
  + what the response is
- plug-in mechanism
  + Make the generic code accept a callback module as an argument
  + Maintain the module atom in the process state
  + Invoke callback-module functions as needed

---

## Open telecom platform (OTP)
- Provides patterns and abstractions for tasks including:
  + creating components
  + building releases
  + developing server processes (`gen_server`)
  + handing and recovering from runtime errors
  + logging
  + event handling
  + upgrading code

---

## OTP behaviours
- The generic logic that implements a common pattern

#### Behaviour & Callback modules
- Behaviour modules
  + The generic logic that accepts a callback module
  + Define a contract
- Callback modules
  + Implement the specifics

#### Some predefined behaviours that OTP ships
- `gen_server` - generic implementation of a stateful server process
- `supervisor` - provides error handling and recovery in concurrent systems
- `application` - generic implementation of components and libraries
- `gen_event` - provides event-handling support
- `gen_fsm` - runs a finite state machine in a stateful server process

---

## The `gen_server` behaviour
- For production, use the `gen_server` behaviour instead of re-inventing a generic server.

#### Features
- support for calls and casts
- customizable timeouts for call requests
- propagation of server process crashes to client processes waiting for a response
- support of distributed systems

#### Callbacks
- Six callbacks are required

---

## Elixir's GenServer module

```elixir
defmodule KeyValueStore do
  use GenServer
  //...
```

```elixir
iex> KeyValueStore.__info__(:functions)
[
  code_change: 3,
  handle_call: 2,
  handle_call: 3,
  handle_cast: 2,
  handle_info: 2,
  init: 0,
  init: 1,
  terminate: 2
]
```

#### Call and cast requests
- Implement `handle_call/3` callback: gen_server will send the message with `:"$gen_call"` atom.
- Implement `handle_cast/2` callback: gen_server will send the message with `:"$gen_cast"` atom.

#### Handling custom messages (ones that are not specific to gen_server)
- Send a message by calling `:timer.send_interval/2`.
- Implement `handle_info/2` callback.
- NOTE: Whenever we override `handle_info/2` callback, we must provide the match-all clause that does nothing.

#### Alias registration
- We can provide the process alias as an option to [GenServer.start/3](https://hexdocs.pm/elixir/GenServer.html#start/3).

```elixir
def start do
  GenServer.start(
    KeyValueStore,             # Callback module atom
    nil,                       # Initial params
    name: :my_key_value_store) # Alias
end

def put(key, value) do
  GenServer.cast(:my_key_value_store, { :put, key, value })
end

def get(key) do
  GenServer.call(:my_key_value_store, { :get, key })
end
```

#### Stopping the server
- Return `{:stop, reason, new_state}` from `handle_*` callbacks.
- Before the process is terminated, `terminate/2` is invoked, in which we can perform cleanup.

#### Process lifecycle

```
  Client process                       Server process
================================================================================
                    spawn
GenServer.start --------------> gen_server init -----------> init
                                       |
                                       |
GenServer.cast ------                  |             ------> handle_cast
                    |  messages        V             |
GenServer.call <--------------- gen_server loop -----------> handle_call
                    |                  |             |  
send ----------------                  |             ------> handle_info
                                       |
                                       |-------------------> terminate
```

---

## OTP-compliant processes
- Avoid spawing plain processes.
- Ensure that all our processes are OTP-compliant processes

#### Benefits
- Can be used in supervision trees
- Errors in processes will be logged with details.
