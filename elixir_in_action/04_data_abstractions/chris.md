# Data abstractions

In Elixir, higher-level data structures are implemented with pure, stateless modules.

## TLDR

- Modules are used to create data abstractions
- Maps can be used to group different fields together
- Structs are a special type of maps that are coupled to a module
- Polymorphism can be implemented with protocols

## The Details

Elixir promotes decoupling of data from code. Instead of calling methods on objects,
you call module functions and provide data as input arguments.

- _Modifier functions_ return data of the same type 
- _Query functions_ return some piece of information from the data

> **CONVENTION**: Module functions usually expect an instance of the data
  abstraction as the first argument. Using the `|>` operator to chain operations
  together is possible due to this convention.

### 4.1 Abstracting with modules

### 4.1.1 Basic abstraction

### 4.1.2 Composing abstractions

### 4.1.3 Structuring data with maps

### 4.1.4 Abstracting with structs

- `defstruct` macro defines a struct
- Structs allow you to specify an abstraction and bind it to a module
- A struct may only exist in a module
- Each module can only define one struct
- Pattern matching with structs works similarly to maps
- A benefit of pattern matching is that input type is enforced

**Structs vs. maps**

- Structs are in reality just maps with extra metadata
- The `__struct__` key/value pair is a part of each struct and
  is used to perform proper runtime dispatches within polymorphic code
- A struct pattern can't match a map, but a map pattern can match a struct

**Records**

- Records can be defined with `defrecord` and `defrecordp` macros
  in the `Record` module
- Records are mostly present for historical reasons (before maps)
- Some Erlang libraries use records, so they need to be imported
  and defined in Elixir as a record

### 4.1.5 Data transparency

### 4.2 Working with heirarchical data

### 4.2.1 Generating IDs

### 4.2.2 Updating entries

### 4.2.3 Immutable heirarchical updates

### 4.2.4 Iterative updates

### 4.3 Polymorphism with protocols

Polymorphism is a runtime decision about which code to execute based on the
nature of the input data. In Elixir, this is done using _protocols_.

### 4.3.1 Protocol basics

A _protocol_ is a module where you declare functions without implementing them.
If the protocol isn't implemented for the given data type, an error is rased.

### 4.3.2 Implementing a protocol

Protocols are implemented using the `defimpl` macro.

```elixir
defimpl String.Chars, for: Integer do
  def to_string(thing) do
    Integer.to_string(thing)
  end
end
```

The type is an atom and can be and of the following aliases:
`Tuple`, `Atom`, `List`, `Map`, `Bitstring`, `Integer`, `Float`, `Function`,
`PID`, `Port`, or `Reference`.

In addition, the alias `Any` is allowed.

The protocol implementation doesn't have to be part of any module.
You can place the protocol implementation anywhere in your code.

> **NOTE**: Protocol dispatch may be significantly slower than a function
  call due to the way protocols work. Elixir optimizes this with protocol
  consolidation, which analyzes your project and compiles the dispatch map.

### 4.3.3 Built-in protocols

- Implementing the `Access` protocol for your structure allows you to use
  `[]` syntax
- Implementing the `Enumerable` protocol lets you take advantage of the
  `Enum` and `Stream` modules for free
- Implementing the `Collectable` protocol allows you to use your structure
  in comprehensions
