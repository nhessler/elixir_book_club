# Sharing State

a.k.a. "using ETS tables for glory and profit"

## TLDR

- ETS is a mutable data structure for sharing state
- ETS tables reside in a separate memory space
- Key-based lookups are very fast
- Non-key lookups should use _match patterns_ or _match specifications_
- When sharing data among many processes, ETS can help performance and scaling

## The Details

### 10.1 Single-process bottleneck

- When you pass a lambda to another process, all external data referenced
  by the lambda is deep copied

### 10.2 ETS tables

Characteristics of ETS tables:
- No specific ETS data type
- ETS tables are mutable
- Multiple processes can write to or read from a single ETS table
- Minimum isolation is ensured
- An ETS table resides in a separate memory space
- An ETS table is linked to its owner process
- No automatic garbage collection

Inside BEAM, ETS tables are powered by C code for performance.

### 10.2.1 Basic operations

ETS table types:
- `:set` is the default and allows one row per key
- `:ordered_set` is like `:set` but rows are sorted in term order
- `:bag` allows multiple rows with the same key
  (but two rows can't be identical)
- `:duplicate_bag` is like `:bag` with duplicates

ETS access permissions:
- `:protected` is the default. 
  owner process can read/write.
  other processes can only read.
- `:public` allows all processes to read/write to the table
- `:private` allows only the owner process to access the table

If you provide a `:named_table` option, the table can be accessed
like a locally registered process alias:

```elixir
:ets.new(:my_table, [:named_table])
:ets.insert(:my_table, {:motto, "Elixir is fun!"})
:ets.lookup(:my_table, :motto)
```

# 10.2.2 ETS powered page cache

> **CONVENTION**: Create the cache table in an existing supervisor
  to avoid creating dummy processes.  The table should be named so
  that you can access it from unrelated processes in the supervision
  tree.

- Try to run independent operations in separate processes.

Use _match patterns_ and _match specifications_ to fetch specific
values from rows
- `:ets.match_object/2` accepts a _match pattern_ that describes
  the shape of the row
- when you don't care about a tuple, you must pass the `:_` atom instead
  of the typical match-all anonymous variable `_`

Match specifications consist of:
- *head* - the rows you want
- *guard* - filters
- *result* - the shape of the returned data

Using `:ets.fun2ms/1`
- `:ets.fun2ms/1` is a pseudo function that runs at compile time
- use `@compile {:parse_transform, :ms_transform}` to enable it
- it cannot be invoked at runtime
- lambdas cannot be passed to it

### 10.2.4 Other use cases for ETS

- Public ETS table can be used to preserve state across process restarts
- Persisting state in ETS should only be used for the error kernel
- Data is copied between ETS tables and client processes

### 10.2.5 Beyond ETS

- [Disk-Based ETS](http://erlang.org/doc/man/dets.html)
- Erlang ships with [Mnesia](http://erlang.org/doc/man/mnesia.html)
