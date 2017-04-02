# 9 Isolating Error Effects

## 9.1

* An important part of a `one-for-one` Supervisor is confining an errors impact to a single worker and take corrective measures only on that process.
* `{:via, module, term}` tuples are used for defining where a process has been registered
* 4 methods needed to build a registry
  * register_name
  * unregister_name
  * whereis_name
  * send (don't forget to add `import Kernel, except: [send: 2]` to remove Kernel.send/2)
* add an ':id` to each worker that needs a unique id for a supervisor to track it.
* Supervision trees are a nested structure of workers and supervisors
* Supervisors should only start OTP compliant processes
  * GenServer
  * Supervisor
  * Agent
  * Task
* If a worker needs to do some cleanup on termination setup an exit trap in the `init/1` callback
* Supervisors can specify a shutdown strategy for children
  * integer (time to allow child for graceful shutdown)
  * :infinity (wait forever for child to shutdown)
  * :brutal_kill (don't allow graceful shutdown)
* Supervisors can supervise temporal processes as well (Think http socket process)
 * `restart: :temporary` will not try to restart process
 * `restart: :transcient` will only restart if shutdown abnormally

## 9.2

* `:simple_one_for_one` supervisors
  * No child is started upfront
  * there can be multiple children but they are all started by the same function
* dynamically starting child processes in a supervisor could lead to race conditions. be careful!
* Todo.Cache remains a GenServer backed process to avoid race conditions.
  * remember code in a function runs synchronously.
  * since Todo.Cache is it's own process, only one `handle_call` will be run at a time.
* Supervisor Strategies
  * :one_for_one
  * :simple_one_for_one
  * :one_for_all
  * :rest_for_one

## 9.3

* *Let It Crash* is not the same as *Let Everything Crash*
  * Deal with critical processes that shouldn't crash
  * Deal with errors that can be handle in a meaningful way
* Keep really important code as small and clean as possible
  * consider splitting out complex logic from critical state
  * possible use of `try/catch` to avoid process crashes
* even with protections measures in place there should still be supervision setup
