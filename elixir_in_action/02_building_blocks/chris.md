# Building Blocks

"A somewhat long, not particularly exciting tour of language features."

## TLDR

- Elixir code is divided into modules and functions
- Elixir is a dynamic language
- Data is immutable, but variables can be rebound
- Primitive data types: `numbers`, `atoms`, and `binaries`
- There is no boolean type
- There is no nullability
- There is no string type
- The only complex types are tuples, lists, and maps
- Elixir adds `Range`, `keyword lists`, `HashDict`, and `HashSet` as 
  abstractions on top of the Erlang data system
- Functions are first-class citizens
- Module names are atoms that correspond to `.beam` files on disk
- `iex`, `elixir`, and `mix` are the primary Elixir tools

## The Details

### 2.1 The interactive shell

- `iex` starts an instance of the BEAM and the interactive Elixir shell
- Similar to `irb` in the Ruby world

### 2.2 Working with variables

- Elixir is a dynamic programming language
- Each expression in Elixir has a result
- Variable names start with a lowercase alphabet character or underscore

> **CONVENTION**: Variables use snake_case naming.

- Variable names can end with `?` and `!`

> **CONVENTION**: Only use `?` and `!` when naming functions.

- Variables are mutable, but the data they point to is immutable

### 2.3.1 Modules

- Every Elixir function must be defined inside a module

> **CONVENTION**: Filenames use `.ex` extension.

- Modules must be defined in a single file, but a single file can contain
  multiple module definitions
- Module names can contain a `.` (dot), but it is merely a syntactic
  convenience and does not imply hierarchy between modules

> **CONVENTION**: Modules use CamelCase naming.

### 2.3.2 Functions

- Functions are always part of a module

> **CONVENTION**: Ending a function name with `!` indicates the function
may raise a runtime error.

- `defmodule` and `def` are **macros** and not keywords
- Short functions can be defined in a single line using a condensed form

```elixir
defmodule Math do
  def add(a, b), do: a + b
end
```

- The `|>` (pipeline) operator makes function chaining easy to read
  - It places the result of the previous function as the first argument
    of the next function

> GOTCHA: Multiline pipelines don't work in the shell because the first
  line is a complete expression and hitting return will evaluate it immediately.

### 2.3.3 Function arity

- `\\` operator allows you to specify default arguments
- A single function cannot accept a variable number of arguments

### 2.3.4 Function visibility

- `def` is used to define a public function
- `defp` exists to define a private function

### 2.3.5 Imports and aliases

- Importing a module allows calling its public functions without prefixing
  them with the module name
- `Kernel` module is automatically imported into every module
- Alternative to `import` is `alias`, which allows you to reference a 
  module using a different name

```elixir
alias IO, as: MyIO
```

### 2.3.6 Module attributes

- Module attributes are similar to annotations in other languages
- `@moduledoc` and `@doc` are useful attributes for generating documentation
- `@spec` attribute can be used to define typespecs

### 2.3.7 Comments

- Comments in Elixir start with `#`
- Elxir doesn't have block comments

### 2.4 Understanding the type system

Elixir uses the Erlang type system.

### 2.4.1 Numbers

- Integers or floats
- `/` (division) operator always returns a float
- Integer division can be performed with `Kernel.div`

> **SYNTACTIC SUGAR**: The underscore character can be used in numbers for 
  readability.

```elixir
iex(1)> 1_000 + 2_034
3034
```

### 2.4.2 Atoms

- Atoms are literal named constants, similar to symbols in Ruby or LISP
- Atoms consist of two parts: the _text_ and the _value_

> **SYNTACTIC SUGAR**: Atom constants can be defined without colons if the atom
  name starts with an uppercase letter. This is an alias and is transformed
  into an atom at compile time.

- Elixir doesn't have a dedicated boolean type
- Under the hood, `true` and `false` are implemented as atoms!

```elixir
iex(1)> true == :true
true
```

- `:nil` is a special atom that is similar to null in other languages
- Like `true` and `false`, you can reference `nil` without a colon

### 2.4.3 Tuples

- Tuples are untyped structures (or records)
- They're often used to group a fixed number of elements together

### 2.4.4 Lists

- Lists operate like a singly linked list
- Therefore, almost all list operations require a full list traversal
- To append, you can use a negative value (-1) for the insert position
- `++` operator concatenates two lists
- Internally, lists are recursive structures of (head, tail) pairs
- Lists are great for recursive implements that do something on the
  `head` (first element) and do something else on the `tail` because
  both operations are O(1)

### 2.4.5 Immutability

- Elixir data can't be mutated
- Every function returns a new, modified version of the input data
- Variables CAN be rebound to new data
- Functions can still have side effects

### 2.4.6 Maps

- A map is a key-value store implemented in Erlang/OTP 17.0+
- Maps don't perform well for large numbers of elements.
  In that case, use `HashDict` instead
- You can only modify values that already exist in the map.
  `Map.put/3` lets you insert a new key/value pair
- `Dict` module can be used as a wrapper on top of any abstract
  key-value structure

### 2.4.7 Binaries and bitstrings

- Binaries are sequences enclosed by `<<` and `>>` operators
- You can concatenate two binaries or bitstrings with the `<>` operator

### 2.4.8 Strings

- Elixir doesn't have a string type
- Strings are represented as a binary or list type
- Binary strings and character lists aren't compatible
- Binary strings are generally preferred over character lists

Binary strings are defined between double-quotes.

- `#{}` supports embedded string expressions
- `~s()` is _sigil_ syntax and allows quotes
- `~S()` is another _sigil_ form that does not interpolate or escape
- _heredocs_ syntax are wrapped with `"""` on their own lines and 
  allow better formatting for multi-line strings
- `<>` concatenates strings (because they're binaries!)

Character lists are defined between single-quotes.

- Character lists are basically linked lists of integers that represent characters
- When a list consists of integers that represent printable characters,
  it is printed to the screen as a string.

### 2.4.9 First-class functions

- Functions are first-class citizens, so they can be assigned to variables 
  and passed to other functions
- Anonymous functions can be assigned to a variable

> **GOTCHA**: Elixir requires a dot operator (`.`) to make invoking anonymous 
  functions explicit.

```elixir
iex(1)> square = fn(x) -> x * x
...(1)> end
#Function<6.52032458/1 in :erl_eval.expr/5>
iex(2)> square.(2)
4
iex(3)> square(2)
** (CompileError) iex:3: undefined function square/1
```

- The `&` (_capture_) operator takes a full function qualifier and turns 
  it into a lambda
- Lambdas can reference any variable from the outside scope
- Closures always capture a specific memory location

### 2.4.10 Other built-in typespecs

- _references_ are an _almost_ unique piece of information in a BEAM instance,
  but after you restart the BEAM instance, reference generation starts over
- _pid_ is used to identify an Erlang process
- _port identifier_ is a mechanism used in Erlang to talk to the outside world

### 2.4.11 Higher-level types 

- Elixir provides higher-level abstractions on top of the Erlang type system
- The most common are `Range`, `Keyword`, `HashDict`, and `HashSet`
- _Enumerables_ are an abstract collection you can iterate over
- _Collectables_ are an abstract collection you can put elements into

Ranges are abstractions for a range of numbers.

- Internally, they are represented as a map with range boundaries (not a list)

Keyword lists are a special case of lists where each element is a 
two-element tuple and the first element is an atom.

- Useful to allow clients to pass an arbitrary number of optional arguments
- Elixir allows you to omit the square brackets if the last argument is a 
  keyword list

HashDict is a module that implements an arbitrary sized key-value lookup
structure.

- HashDict is more performant than map/keyword list for larger collections
- Ordering is not guaranteed

HashSet is an implementation of a set.

### 2.4.12 IO lists

An IO list is a special type of list used for building output that will be 
fowarded to I/O.

Each element of an IO list must be one of the following:

- Integer in range 0..255
- A binary
- An IO list

This definition means that IO lists are very deeply nested structures where
leaf elements are plain bytes.

### 2.5 Operators

- `==` (weak eq) and `===` (strong eq) are relevant when comparing integers
  to floats
- Many operators in Elixir are actually functions in the Kernel module

### 2.6 macros

- Macros are bits of Elixir code that can change the semantics of input code
  at _compile-time_
- Many parts of Elixir are written in Elixir through macros

### 2.7.1 Understanding the runtime 

- When you start BEAM with Elixir tools, some code paths are predefined
  (`-pa` switch)
- In Erlang, modules correspond to atoms
- At runtime, module names are atoms, which map to compiled _xyz.beam_
  files (where _xyz_ is the expanded form of a module alias)
- `Kernel.apply/3` can be useful to make a runtime decision about what
  function to call

### 2.7.2 Starting the runtime

- `iex` takes input, **interprets** it, and prints the result.
  Interpreted code won't be as performant as the compiled code
- `elixir` can be used to run a single Elixir source file

> **CONVENTION**: Elixir scripts end with `.exs` extension.

- If you don't want a BEAM instance to terminate, use the `--no-halt`
  parameter.  This is useful if your main code simply starts up concurrent
  tasks in individual processes
- `mix` is used to manage projects made up of multiple source files
