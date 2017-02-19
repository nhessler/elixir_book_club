# 6 Generic Server Processes

## 6.1

* Basic Tasks of any Server Process
  * Spawn a separate process
  * run an infinite loop
  * Manage the state appropriately
  * React to messages
  * Send responses back to callers
* The `ServerProcess` and callback-module feels very OO in how they interact.
* `TodoServer` using `ServerProcess`
  * doesn't feel lighter
  * it is easier to read
  * Added default register process to callback module name
  * Added timeout in `ServerProcess.call`

## 6.2

* OTP Behaviours
  * gen_server
  * supervisor
  * application
  * gen_event
  * gen_fsm
* Some of the differences in `ServerProcess` and `GenServer` are a result of using `spawn_link` and/or `spawn_monitor`. Worth checking out if you feel so inclined.
* `handle_info` is for requests that don't fit the standard `cast` or `call` request structures.
* Is it true that any function defined locally nulifies functions included in a `use`? Or, is this specific to `GenServer`?
* GenServer and Erlang are *Actor Like* implementations but good to know it's not exact or even directly inspired in either direction
