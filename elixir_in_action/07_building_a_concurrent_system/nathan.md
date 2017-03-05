# 7 Building A Concurrent System

## 7.1

* pretty standard project structure. similar to Ruby.
* `iex -S mix` is awesome!
* moved project to umbrella project.

## 7.2

* processes run concurrently, code in a process runs sequentially.
* high message volume processes should have very quick action sequences

## 7.3

* casts increase scalability of an app but come with a few downsides.
  * no knowledge of whether message reached target
  * no knowledge of whether actions completed successfully
* long running inits can even happen a couple levels deep as you build out your app
* when delaying init code it is only guaranteed to be first if process is unregistered
* always be concerned with process message volume and action/response time
* reasons for running code in a dedicated process
  * must manage long-living state
  * handles a resource that can and should be reused (e.g. TCP, DB connection, file handler)
  * critical section of code must be synchronized

## 7.4

* GenStage? Flow? do these help with some of these examples?
