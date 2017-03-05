# 8 Fault Tolerance Basics

## 8.1

* Error Types
  * errors (raise)
  * exits
  * throws
* One way to deal with these errors is in a `try...catch`
* `catch` uses Pattern Matching to do it's catching similar to `receive` and `case` and therefore allows for multiple clauses
* `try...catch` also comes with an `after` option that will run regardless of an error being thrown or not
* use `try...cach` carefully. do not include in methods that need to be tail recursive as it will break that requirement
* *Let It Crash*

## 8.2

* processes are isolated and protected because memory is not shared!
* `{:exit, :normal}` is sent by any process that exits normally
* `Process.link/1` creates bidirectional links from current process to any other process
* a process can have multiple links and will send an exit signal to all of them.
* To setup a trap you call `Process.flag/2`
* `Process.monitor/1` creates unidirectional links from current process to target process
* `Process.demonitor/1` removes unidirectional link from current prcess to target process
* monitors and links are different in 2 ways
  * monitors are unidirectional and links are bidirectional
  * linked processes fail when other process fails, monitor does not.
* A monitor is setup in `Genserver.call`

## 8.3

* `worker` and `supervise` both create tuples. the tuple is passed back to `Supervisor` to actually get the processes started
* note that "restart" really means start a new process with same functionality when one crashes. it does not mean actually restart the same process which isn't possible.
* Supervisors can be started without a callback module.
* `Process.whereis` and `Process.exit` are useful. Guessing I should be more familiar with the `Process` module
* **Beware dangling processes**
* Supervisors will only restart child processes a certain # of times in a given interval then fail themselves
