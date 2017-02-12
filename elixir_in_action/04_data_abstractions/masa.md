# Elixir in Action - Chapter 4 - Data abstraction

- Polymorphism can be implemented with protocols.
- A protocol defines an interface that is used by the generic logic

---

## Basic principles
- Stateless modules
- Pure functions:
  + Usually expect an instance of data abstraction as the first argument
  + Modifier functions: return a modified version of the abstraction
  + Query functions: return some other type of data

---

## Structs
- Special kind of maps
- May exist only in a module
- A single module can define only one struct
- Cannot call the Enum function on a struct

```elixir
defmodule Fraction do
  defstruct a: nil, b: nil  # A struct may exist only in a module.

  def new(a, b) do
    %Fraction{a: a, b: b}
  end

  def value(%Fraction{a: a, b: b}) do
    a / b
  end

  def add(f1, f2) do
    new_a = f1.a * f2.b + f2.a * f1.b
    new_b = f1.b * f2.b
    new(new_a, new_b)
  end
end
```

```elixir
## Creating an instance and calling some module functions on it.
iex(16)> half    = Fraction.new(1, 2)  # %Fraction{a: 1, b: 2}
iex(17)> quarter = Fraction.new(1, 4)  # %Fraction{a: 1, b: 4}

## We cannot call Enum functions on a struct.
iex(20)> Enum.to_list(half)  # ** (Protocol.UndefinedError)

## A struct is a special map with the key __struct__.
iex(19)> Map.to_list(half)  # [__struct__: Fraction, a: 1, b: 2]
# NOTE: We should not rely on the data abstraction's internal structure.
```

#### Matching a struct to a map

```elixir
## We can match a struct to a map.
iex(20)> %{a: a, b: b} = half  # %Fraction{a: 1, b: 2}
iex(21)> a                     # 1
iex(22)> b                     # 2
```

#### Inspecting every line by inserting `IO.inspect/1`
- `IO.inspect/1` prints the data structure and returns the same data structure unchanged

```elixir
Fraction.new(1, 4)               |>
IO.inspect |>
Fraction.add(Fraction.new(1, 2)) |>
IO.inspect |>
Fraction.add(Fraction.new(1, 4)) |>
IO.inspect |>
Fraction.value
# %Fraction{a: 1, b: 4}
# %Fraction{a: 6, b: 8}
# %Fraction{a: 32, b: 32}
# 1.0
```

#### Data transparency
- The entire data structure is always visible
- Clients can inspect the entire structure but should not rely on it

```elixir
iex(25)> inspect(half)                  # "%Fraction{a: 1, b: 2}"
iex(26)> inspect(half, structs: false)  # "%{__struct__: Fraction, a: 1, b: 2}"
```

---

## Working with hierarchical data

#### Updating a deep hierarchical record
- 1. walk down the tree to the particular part that needs to be modified
- 2. transform it and all of its ancestors

#### The macros for elegant deep hierarchical updates
- relies on the Access protocol
- `put_in/2`, `get_in/2`, `update_in/2` etc cannot be built dynamically
- For runtime use, use equivalent functions that accept data and path as separate arguments, such as `put_in/3`

---

## Records
- Used to be one of the main tools for structuring data before map appeared

---

## Polymorphism with protocols

> Polymorphism is a runtime decision about which code to execute, based on the nature of the input data.

- can be done by using the language feature protocols
- protocol must be implemented for each data type that we want the function to use.
- built-in protocols: `Enumerable`, `Collectable`, `String.Chars`, `List.Chars`, etc

```elixir
# Enum.each accepts data types that implements String.Char and Enumerable protocols.
Enum.each([1, 2, 3], &IO.puts/1) # List
Enum.each(1..3,      &IO.puts/1) # Range
```

```elixir
# Trying to call IO.puts/1 on a user-defined type that does not implement String.Chars protocol.
iex(22)> IO.puts(todo_list)
# ** (Protocol.UndefinedError) protocol String.Chars not implemented for %TodoList{...}
```

#### Implementing a protocol

```elixir
# Implementing String.Chars protocol for a user-defined type
defimpl String.Chars, for: TodoList do
  def to_string(thing) do

    # TODO: do something

  end
end
```

```elixir
# Implementing Collectable protocol for a user-defined type
defimpl Collectable, for: TodoList do
  def into(original) do
    {original, &into_callback/2}  # A tuple of original data and appender lambda.
  end

  # Implement the appender.
  defp into_callback(todo_list, {:cont, entry}) do
    TodoList.add_entry(todo_list, entry)
  end

  defp into_callback(todo_list, :done), do: todo_list
  defp into_callback(todo_list, :halt), do: :ok
end
```
