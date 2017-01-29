# Chapter 2. Building Blocks

> Everything in Elixir is an expression that has a return value.

```elixir
Interactive Elixir (1.4.0) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> System.halt
```

> Data in Elixir is immutable: its contents can't be changed. But variables can be rebound to a different value.

I find this to be harder than Erlangs:

```erlang
Eshell V8.2  (abort with ^G)
1> X = 2.
2
2> X = 1.
** exception error: no match of right hand side value 1
3>
```
And the whole point about variables being mutable and data to which they point not being so just means I can sneak in bad practices.

## Modules

Module names start with upper case and are written in CamelCase style. Dot(.) is used for organization, and if modules are nested can be used to reference a child module: `ParentModule.ChildModule`

## Functions

Names start with lowercase letter or underscore and are followed by a combination of alphanumerics and underscores.

`func?` is used to indicate that func returns true or false.
`func!` is used to indicate that a runtime error might be raised.

Function without arguments should not have parentheses in the definition.

shorthand definition:

```elixir
def well_hello, do: IO.puts "Hello!"
```

Note: parentheses are optional in Elixir, so omitting them would look like:

```elixir
Module.some_func 1, 2
```

Functions with same name and different arities should be treated as different functions all together.

Delegation between functions where lower arity function is used as an "entry level" with cleaner argument list to the higher arity function is simplified in Elixir with `\\` which denotes a default argument.

Definitions that use `\\` simply generate functions with multiple arities.

`def` vs `defp` is used to mark a function as exported, which really means that it is accessible from the outside vs inside of the module.

`import` ing a module allows you to call its functions without the module name.

`alias` allows you to reference a module under a different name.
Okay, this one is a confusing... since
```elixir
alias ModuleName, as: MN
```
`!NB` is arguably not very helpful.
Now, Phoenix has a lot of the `alias AppName.Model` which really does more of a `alias AppName.Model, as: Model` and is a confusing shorthand especially when you see all four `use`, `import`, `alias`, and `require` together :S

from docs
```elixir
# Alias the module so it can be called as Bar instead of Foo.Bar
alias Foo.Bar, as: Bar

# Ensure the module is compiled and available (usually for macros)
require Foo

# Import functions from Foo so they can be called without the `Foo.` prefix
import Foo

# Invokes the custom code defined in Foo as an extension point
use Foo
```

```elixir
defmodule Example do
  use Feature, option: :value
end
```
is compiled into
```elixir
defmodule Example do
  require Feature
  Feature.__using__(option: :value)
end
```

`!NB` Module attributes are awesome :)

```elixir
@var 123
```
is a compile time constant and can be queried in runtime!

`!NB` `elixirc circle.ex` results in `Elixir.Circle.beam`

Typespecs allows one to provide type specification which can then be analyzed by Dialyzer.

```elixir
@pi 3.14

@spec area(number) :: number
def area(r), do: r*r*@pi
```

> There is no upper limit on the integer size, and they can be arbitrarily large numbers

`!NB` would be fun to implement a type using lists like
[123,456,789] to denote 123456789

## Atoms

An atom consists of two parts: the text and the value. At runtime the text is kept in the atom table. The value that goes into the variable references the atom table.

`variable = :some_atom` the variable contains just the reference to the atom table, making it efficient for storing constants.

You an omit the colon and start with upper case. In addition at compile time it gets expanded to :"Elixir.AnAtom"

```elixir
iex(7)> AnAtom == :"Elixir.AnAtom"
true
```

Boolean type is also represented by atoms `:true` and `:false`. `:nil == nil`.

Atoms nil and false are _falsy_ while everything else is _truthy_.

The operator `||` returns the first expression that is not _falsy_.
If all expressions evaluate to a falsy value, the last expression is returned.

The operator `&&` returns the second expression but only if the first expression is truthy.

## Tuples

`Kernel.elem/2` is used to extract elements from a tuple.
`Kernel.put_elem/3` is used to change the value in the tuple.

## Lists

Pretty much cons cells. 'nuf said.

```lfe
lfe> (cons 1 (cons 2 ()))
(1 2)
```

```elixir
iex(12)> [1 | [2 | []]]
[1, 2]
iex(13)> hd [1, 2, 3]
1
iex(14)> tl [1, 2, 3]
[2, 3]
```

```lfe
lfe> (cadr (1 2 3))
2
```

```elixir
iex(1)> hd tl [1, 2, 3]
2
```
Tuples are copied and copies are shallow.
Lists are shallow copied to the diff, followed by the diff and followed by the shared tail.

Adding elements to the end of the list is thus expensive as you shallow copy the whole list.

If you add to the head, the new list doesn't copy anything, the old list becomes the tail.

Immutability gives an ability to perform atomic in-memory operations. Thus, when working with data, you can always roll back to where you started.

## Map

Introduced in OTP 17.0 and does not perform well with large number of elements. What about 19?
Being a key value store it allows accessing elements by name.

```elixir
iex(3)> bob = %{:name => "Bob"}
%{name: "Bob"}
iex(4)> jane = %{name: "Jane"}
%{name: "Jane"}
iex(5)> bob["name"]
nil
iex(6)> bob[:name]
"Bob"
iex(7)> bob.name
"Bob"
iex(8)> jack = %{"name" => "Jack"}
%{"name" => "Jack"}
iex(9)> jack.name
** (KeyError) key :name not found in: %{"name" => "Jack"}
iex(9)> jack["name"]
"Jack"
iex(10)> older_bob = %{bob | name: "Older Bob"}
%{name: "Older Bob"}
iex(11)> map = %{data: %{value: 1}}
%{data: %{value: 1}}               
iex(12)> put_in(map, [:data, :value], 2)
%{data: %{value: 2}}
```

## Binaries and Bitstrings

Binary is a chunk of bytes. User << and >> operators to create.

## Strings

Elixir does not have a dedicated string type. Represented by binary or list type.

You can use _sigils_ to declare strings as well `~s()`. Use `~S()` is you don't want values to be interpolated.
_heredocs_ syntax for multiline Strings involves starting and ending with `"""`

Since strings are binaries, use `<>` to concat.

## Character Lists

```elixir
iex(13)> 'ABC'
'ABC'
iex(14)> [70,71,72]
'FGH'
```

Similar `~c()`, `~C()`, and ''' '''

Less efficient compared to Strings since these are lists.

## First class functions

Anonymous function (lambda)

```elixir
iex(15)> square = fn(x) -> x * x end
#Function<6.52032458/1 in :erl_eval.expr/5>
iex(16)> square.(5)
25
```

`&` _capture operator_ turns module name, function name and arity into a lambda
`&IO.puts/1`
```elixir
iex(18)> 1..3 |> Enum.each(&IO.puts/1)  
1
2
3
```
## Closures

A lambda can reference any variable from outside the scope and the memory location occupied by that value won't be garbage collected.

## Other Types

* Reference - unique piece of information in BEAM
* pid - process identifier
* port identifier - communication to external programs is done through ports

## High level types

* Range - 1..5 (enumerable)
* Keyword List - list of two element tuples where first element is an atom [{monday: 1}]
* HashDict - module implementing abritrary sized key-value lookup structure. (enumerable and collectable)
* HashSet - is an implementation of a set. (enumerable)
* IO Lists - a special list to be incrementally built for forwarding to I/O device.

## Operators

Many are actually functions.
`a + b` is `Kernel.+(a,b)`

## Macros

More LISP goodness. Just don't use them when you can solve things by writing a function...
Operates and transforms code at compile time. `unless` and `if` are examples.

Instead of keywords, Elixir uses LISP approach and has a smaller core.

## Runtime

`defmodule Geometry do` is `:"Elixir.Geometry"` and is `Elixir.Geometry.beam` on disk.

The `iex` tool performs in-memory generation of bytecode and loads the modules.

Could define pure Erlang modules via :module_name (not recommended)

`Kernel.apply/3` allows calling functions at runtime -> `apply(IO, :puts, ["test"])`

can pass `--no-halt` to `elixir` to prevent the BEAM instance from being stopped.

`mix` tool is definitely my friend...
