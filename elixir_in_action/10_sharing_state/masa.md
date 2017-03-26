# Elixir in Action - Chapter 10 - Sharing state

## Erlang Term Storage (ETS)
- A mutable data structure where you can store tuples.
- Resides in a separate memory space and can be accessed concurrently by many processes.
- Enables us to share the system-wide state without introducing a dedicated server process.
- Key-based lookups in ETS tables are very fast.
- Be careful about concurrent writes, and try to serialize writes to the same table (or to the same row) through the single process.
- At most 1400 tables per BEAM instance, by default

### When ETS table is useful?
- When many processes depend on it, a single stateful server process may become a bottleneck.

### Do not overuse ETS
- Processes should be our first choice for maintaining state.
- An ETS table initially takes up a couple of kilobytes of memory.

### Characteristics
- A table is identified by:
  + id (number)
  + global name (atom)
- ETS tables are **mutable**
- Writes and reads are **concurrent**
- Multiple processes can safely writes to the same row of the same table. The last write wins.
- An ETS table lives in a separate memory space.
- Data is **deep-copied** to and from an ETS table.
- The ETS table is reclaimed when its owner process terminates.
- ETS tables are **not gargage-collected** even if it is not referenced at all.

### Erlang `:ets` module
- Contains all functions related to ETS tables
- [http://erlang.org/doc/man/ets.html](http://erlang.org/doc/man/ets.html)

### Table types
- `:set` - Default. One row per distinct key is allowed.
- `:ordered_set`
- `:bag`
- `:duplicate_bag`

### Table access permissions
- `:protected` - Default. The owner process can read from and write to the table. All other processes can read from the table.
- `:public`
- `:private`

### Table rows
- Each row is an arbitrarily-sized tuple.
- Each tuple element can contain any Erlang term.

### Accessing a table

#### A table accessible by id
- Create a table without `:named_table` option.
- The table name is required but does nothing by default.
- We need to pass around the ETS numerical ID.

```elixir
table = :ets.new(:my_table, [])
# 98326
:ets.insert(table, {:name, "Masatoshi"})
# true
:ets.lookup(table, :name)
# [name: "Masatoshi"]
```

#### A table accessible by table name
- Create a table with `:named_table` option.
- We can use the table name like a locally registered alias.

```elixir
:ets.new(:my_table, [:named_table])
# :my_table
:ets.insert(:my_table, {:name, "Masatoshi"})
# true
:ets.lookup :my_table, :name
# [name: "Masatoshi"]
```

### Fine-grained write synchronization

```
                                           Table owner process
                                              │
                (direct cache reads)          v
Client 1 ────────────────────────────────>
Client 2 ────────────────────────────────> ETS table
Client n ────────────────────────────────>
:                                             ^
  │                                           │
  ├────────> operation_1_writer ──────────────┤
  └────────> operation_2_writer ──────────────┘

  (writes are channeled through operation-specific processes)
```
