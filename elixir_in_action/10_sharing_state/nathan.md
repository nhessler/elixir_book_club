# 10 Sharing State

## 10.1

* A single process that has many more clients than cores on a machine can quickly become a bottleneck

## 10.2

* Characteristics for ETS Tables
  * no specific ETS datatype. referenced by ID or atom
  * ETS tables are mutable
  * multiple processes can read or write to a single ETS table
  * minimum isolation is ensured. last write wins
  * ETS tables reside in a separate memory space
  * ETS tables are *deeply connected* to their owner process
  * other than on owner process termination there is no garbage collection of ETS tables
* Types of ETS tables
  * :set           (unique key per row)
  * :ordered_set   (same as :set with rows in term order (<>))
  * :bag           (duplicate keys allowed but not identical rows)
  * :duplicate_bag (same as :bag but allows identical rows)
* Permission types of ETS tables
  * :protected (owner writes, all read)
  * :public    (all write, all read)
  * :private   (owner writes, owner reads)
* It's not uncommon to create ets tables in a supervisor init.
* Common actions are key based
  * insert
  * lookup
  * delete
* You can do non key based lookups as well
  * traversal
  * iteration in the table
  * most preferred is using match_object which uses matching patterns to find rows
* matching patterns for `:ets.select/2`
  * Head - the main matching pattern
  * Guard - additional patterns
  * Result - Shape of returned data
* match specifications can get complex.
  * use `:ets.fun2ms/1` to generate more complex patterns
  * `:ets.fun2ms/1` is an erlang *pseudo* function similar to macros. compiled at runtime.
* DETS - disk-based erlang term storage
* Mnesia - more featureful database option built on top of ETS or DETS based on your need
