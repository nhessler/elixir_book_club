# Elixir in Acton - Chapter 2 - Building blocks

## Elixir runtime
- An OS process (the BEAM instance), where everything in our Elixir program runs
- A program is typically run by calling functions from modules
- BEAM keeps track of all modules loaded in memory
- When a source file is compiled, .beam files are created for each module
- Can dynamically call functions using `Kernel.apply/3`

---

## Starting the program

### Interactive shell (`iex`)
- Starts Erlang and Elixir runtime in a terminal
- To start, run `iex`
- To quit, hit *Ctrl+C* twice  or `Ctrl-\` or invoke `System.halt`
- Can read Elixir source file (e.g., `$ iex some_module.ex`)

### Running scripts (`elixir`)
- Use `elixir` command for running a single Elixir source file
- When `elixir` command is called, the source file is compiled in memory
- Recommended that we use the `.exs` extension for Elixir script files

### Mix tools (`mix`)
- Used for managing projects that are made up of multiple source files

```elixir
$ mix new masatoshi
$ cd masatoshi

# Edit files creating modules for the masatoshi app...

$ mix run           # Start the system and terminate
$ mix run --no-halt # Start the system and does not terminate
$ iex -S mix run    # Start the system and load the interactive shell
```

---

## Modules
- Defined using the `defmodule` macro
- A collection of functions
- A module must be define in a single file
- A single file may contain multiple module definitions
- Use the dot (`.`) character or nest modules, to organize modules
- No hierarchical relations between modules, after compilation
- Module names are atoms that correspond to beam files on the disk

#### Kernel module
- Automatically imported into every module
- [docs](http://elixir-lang.org/docs/stable/elixir/Kernel.html)

#### Importing a module
- Allows us to call a module's public functions without prefixing them with the module name

```elixir
import IO
puts "Elixir is fun"  # Can call puts instead of IO.puts
```

#### Aliasing a module
- Allows us to reference a module under a different name
- Useful when using a long-name module many times
- Can increase ambiguity in our code

```elixir
alias IO, as: MasasIO
MasasIO.inspect "Konnichiwa sekai"  # Can call MasasIO.inspect instead of IO.inspect
```

---

## Functions
- Defined using the `def` or `defp` macro
- Must be defined inside a module
- No explicit return in Elixir
- Arity - the number of arguments a function receives
  + Functions with the same name but different arities are considered as different functions
  + A lower-arity function can delegate to a higher-arity one providing default arguments

---

## Module attributes
- Used as compile-time constants

```elixir
defmodule Maru do
  @pi 3.14
  def menseki(hankei), do: hankei * hankei * @pi
end

Maru.menseki(2)
```

- Can be registered and be accessed in runtime
- Elixir registers some module attributes by default, such as:
  + `@moduledoc`
  + `@doc`
  + [more](http://elixir-lang.org/docs/stable/elixir/Module.html)

```elixir
# To reference the docs that are marked with the @moduledoc and @doc module attributes,
# we need to compile the file.
defmodule Maru do
  @moduledoc """
  Maru-ni kanrensuru mojuuru desu.
  """
  @pi 3.14

  @doc """
  Maru-no menseki-o keisen-shimasu.
  """
  def menseki(hankei), do: hankei * hankei * @pi
end
```

#### Type specification and dialyzer
- We can provide type information for our functions using the `@spec` attributes.
- We can perform static analysis of our code using [dialyzer](http://erlang.org/doc/man/dialyzer.html) (static analysis tool).
- Our code can be more readable with typespecs provided.

```elixir
@spec insert_at(list, integer, any) :: list
```

---

## Lambdas

```elixir
square = fn(x) -> x * x end
square.(2) # 4
```

```elixir
[1,2,3,4] |> Enum.each(fn(x) -> IO.puts(x) end)
[1,2,3,4] |> Enum.each(&IO.puts/1) # shorthand
```

```elixir
my_calc = fn(x, y, z) -> x * y + z end
my_calc = &(&1 * &2 + &3) # shorthand
my_calc.(2,3,4) # 10
```

---

## Macros
- Perform code transformations at compile time
- Reduce boilerplate
- Provide DSL constructs
- A large part of Elixir is written with macros

---

## Primitive data types

### Numbers

- The `/` operator always returns a float value.

```elixir
1 / 3   # 0.3333333333333333
4 / 2   # 2.0
```

- There Kernel functions can calculate the remainder.

```elixir
div(1, 3) # 0
div(4, 2) # 2
rem(1, 3) # 1
```

- No upper limit on the integer size, taking up as much space as needed to accomodate the number.

### Binaries

```elixir
<<255, 255, 255>>  # <<255, 255, 255>>
<<256, 256, 256>>  # <<0, 0, 0>>
```

### Atoms
- Best used for named constants
- Consists of two parts: text and value

```elixir
:an_atom
:"an atom with spaces"
```

```elixir
AnAtom == :"Elixir.AnAtom"
AnAtom == Elixir.AnAtom
```

---

## Boolean and nil

- In Elixir, a boolean is an atom with the value of `true` or `false`.
- Like Ruby, the `nil` and `false` atoms are treated as falsy; everything else, truthy.

```elixir
# Syntactic sugar - colons can be omitted
:true  == true  # true
:false == false # true
:nil   == nil   # true
```

Logical operators (true and/or false)

```elixir
true and false # false
false or true  # true
not false      # true

not :masatoshi # (CompileError)
```

Shortcut operators (truthy and/or falsy)

```elixir
nil || false || 5 || true  # 5
read_cached || read_from_disk || read_from_database
database_value = conection_extablished? && read_data
```

---

## Strings
- Two types:
  + binary (double quotes) - recommended
  + char list (single quotes)

#### Binary strings (`"double-quote string"`)

```elixir
"Embedded expression #{3 + 0.14}"   # "Embedded expression 3.14"
~s(Embedded expression #{3 + 0.14}) # "Embedded expression 3.14"
~S(Embedded expression #{3 + 0.14}) # "Embedded expression \#{3 + 0.14}"
```

```elixir
"
A string can
span across
multiple lines.
"
# "\nA string can\nspan across\nmultiple lines.\n"
```

```elixir
"""
This is a heredoc.
"""
# "This is a heredoc.\n"
```

```elixir
"Hello" <> " " <> "world"
# "Hello world"
```

#### Char lists (`'single-quote string'`)
- Most of the operations from the String module do not work with char lists.
- Some functions may only work with char lists.
- Char lists can be converted to a binary string using `List.to_string/1`

```elixir
'ABC'         # 'ABC'
[65, 66, 67]  # 'ABC'
```

---

## Complex types
- tuples, list and maps
- Enumerables - An abstract collection we can iterate on
- Collectables - An abstract collection we can put elements into

### Tuples

- Best used for grouping a small fixed number of elements together
- Zero-based index

```elixir
person = {"masatoshi", 77}
```

### Lists
- Dynamic variable-sized collection of data
- Most operations have a O(n) complexity
- Can use the [Enum](http://elixir-lang.org/docs/stable/elixir/Enum) module for lists
- Recursive structure of head-tails pairs

```elixir
a_list = [head | tail]
[1,2,3,4] == [1 | [2 | [3 | [4 | []]]]] # true
hd [1,2,3,4]                            # 1
tl [1,2,3,4]                            # [2,3,4]
new_list = [:new_element | a_list]      # This is a O(1) complexity
```

#### Keyword lists
- Lists of two-element tuples
- Best used for small-size key-value structures
- A lookup operation has a O(n) complexity
- Used for passing named options to a function, by convention

#### IO lists
- Each element must be
  1. An integer in the range of 0 to 255
  2. A binary
  3. An IO list
- Appeinding to an IO list is O(1) because we can use nesting

### Map
- Key-value store, like Ruby hash
- For large collections, consider using HashDicts

```elixir
masa = %{"name" => "Masatoshi", "city" => "Washington"}
masa["name"]  # "Masatoshi"
masa['name']  # nil
masa[:name]   # nil
masa["hello"] # nil

# Updating a map
%{masa | "name" => "Masatoshi Nishiguchi"} # an existing key
Map.put(masa, "state", "DC")               # a non-existent key
```

#### Hashsets
- Stores a unique values
- Works similarly to Hashdicts

#### Range
- Internally a map of range boundaries
- Takes very small memory space regardless of the size of a range

```elixir
1..5 |> Enum.each(&IO.puts/1)
```

```elixir
3  in 1..5  # true
-1 in 1..5  # false
```
