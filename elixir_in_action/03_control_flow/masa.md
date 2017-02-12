# Elixir in Action - Chapter 3 - Control flow

## Pattern-matching basic techniques
- `pattern = expression`
- At runtime, left side pattern is matched to the right side term
- A matching expression always returns the right-side term that is matched against

```elixir
iex(4)> {date, time} = :calendar.local_time
{{2017, 1, 22}, {15, 33, 43}}
iex(5)> {year, month, day} = date
{2017, 1, 22}
iex(6)> {hour, minute, second} = time
{15, 33, 43}
iex(7)> minute
33
```

#### Ignoring unnecessary values with the anonymous variable (`_`)

```elixir
iex(12)> {_, {_, minute, _}} = :calendar.local_time
{{2017, 1, 22}, {15, 43, 45}}
iex(13)> minute
43
```

#### Partial matching for a map

```elixir
iex(26)> %{name: name} = %{name: "Masa", id: 39}
%{id: 39, name: "Masa"}
iex(27)> name
"Masa"
```

#### Matching against the contents of the variable

```elixir
iex(15)> chosen_name = "Masa"
"Masa"
iex(16)> {^chosen_name, language} = {"Masa", "Elixir"}
{"Masa", "Elixir"}
iex(17)> {^chosen_name, language} = {"Taro", "Elixir"}
** (MatchError) no match of right hand side value: {"Taro", "Elixir"}
```

#### Matching binaries

```elixir
iex(30)> <<b1, b2, b3>> = <<123, 234, 32>>
<<123, 234, 32>>
iex(31)> <<b1, rest::binary>> = <<123, 234, 32>>
<<123, 234, 32>>
iex(32)> rest
<<234, 32>>
```

```elixir
iex(33)> <<a::4, b::4>> = <<234>> # 1110 1010
<<234>>
iex(34)> a
14 # 1110
iex(35)> b
10 # 1010
```

```elixir
iex(36)> <<a, b, c>> = "ABC"
"ABC"
iex(37)> a
65
```

#### Matching part of the string

```elixir
iex(42)> command = "ping www.google.com"
"ping www.google.com"
iex(43)> "ping " <> url = command
"ping www.google.com"
iex(44)> url
"www.google.com"
```

```elixir
iex(1)> url = "https://somedomain.com/some_entity/12345"
"https://somedomain.com/some_entity/12345"
iex(2)> "https://somedomain.com/some_entity/" <> entity_id = url
"https://somedomain.com/some_entity/12345"
iex(3)> entity_id
"12345"
```

---

## Comparison in Elixir

In Elixir, if things of different types are compared with `<` and `>`, the following rule is applied.
Notice that a number is considered less than a string.

```
number < atom < reference < fun < port < pid < tuple < map < list < bitstring(binary)
```

---

## [Guard expressions](https://hexdocs.pm/elixir/master/guards.html)
- logical expressions that place further conditions on a clause.
- Only limited number of operators and functions are allowed in guards
  + `==`, `<`, `>`, etc
  + `and`, `or`, etc
  + `<>` and `++` with a literal on the left
  + `+-*/`
  + `in`
  + `is_number/1`, `is_atom/1` etc
  + `abs/1`, `elem/2`, `hd/1`, `length/1`, `map_size/1`, etc
- If an error is raised from inside a guard, that guard expression will return `false`
- Can define a default clause that always matches
- [Using Functions in Elixir Guard Clauses](http://keathley.io/2016/04/09/elixir-guard-clauses.html)

```elixir
# A named multi-clause function with guard expressions
defmodule TestNumber do
  def test(x) when is_number(x) and x < 0, do: :negative
  def test(0), do: :zero
  def test(x) when is_number(x) and x > 0, do: :positive
  def test(_), do: {:error, :invalid_input}
end

iex(23)> TestNumber.test(12)
:positive
iex(24)> TestNumber.test(0)
:zero
iex(25)> TestNumber.test(-1)
:negative
iex(26)> TestNumber.test("Hello")
{:error, :invalid_input}
```

```elixir
# A multi-clause lambda with guard expressions
test_number_fun = fn
  (x) when is_number(x) and x < 0 ->
    :negative
  (0) -> :zero
  (x) when is_number(x) and x > 0 ->
    :positive
  (_) -> {:error, :invalid_input}
end

iex(32)> test_number_fun.(12)            
:positive
iex(33)> test_number_fun.(0)
:zero
iex(34)> test_number_fun.(-1)
:negative
iex(35)> test_number_fun.("Hello")
{:error, :invalid_input}
```

#### Custom guard expressions
- [Defining custom guard expressions](https://hexdocs.pm/elixir/master/guards.html#defining-custom-guard-expressions)

```elixir
defmodule MyInteger do
  defmacro is_even(number) do
    quote do
      is_integer(unquote(number)) and rem(unquote(number), 2) == 0
    end
  end
end
```

```elixir
import MyInteger, only: [is_even: 1]

def my_function(number) when is_even(number) do
  # do stuff
end
end
```

---

## Multi-clause functions
- The same-arity functions, each of which has a different definition
- Replaces classical conditional constructs
- At runtime, the clauses are selected using the order in the source code
- Always group the clauses of the same-arity function together

```elixir
defmodule Shape do
  # Define Shape.area/1
  def area({:rectangle, a, b}), do: a * b
  def area({:square, a}),       do: a * a
  def area({:circle, r}),       do: r * r * 3.14

  # Return {:error, reason} to indicate that an error has occurred.
  # NOTE: This only works for Shape.area/1
  def area(invalid_input), do: {:error, {:unknown_shape, invalid_input}}
end

# Called with a valid input.
iex(2)> Shape.area({:rectangle, 2, 3})
6
iex(3)> Shape.area({:square, 2})
4
iex(4)> Shape.area({:circle, 3})
28.26

# Called with an invalid input.
iex(5)> Shape.area({:circle, 3, 5})
{:error, {:unknown_shape, {:circle, 3, 5}}}
# NOTE: Without error-handling clause, an error would be raised.
# iex(5)> Shape.area({:circle, 3, 5})
# ** (FunctionClauseError) no function clause matching in Shape.area/1
#     iex:2: Shape.area({:circle, 3, 5})

# Binding a function to a variable using the capture operator (`&Module.fun/arity`)
iex(5)> fun = &Shape.area/1
&Shape.area/1
iex(6)> fun.({:circle, 3})
28.26
```

#### Polymorphic functions
- Do different things depending on the input type

```elixir
defmodule Polymorphic do
  def double(x) when is_number(x), do: 2 * x
  def double(x) when is_binary(x), do: x <> x
end

iex(40)> Polymorphic.double(64)
128
iex(41)> Polymorphic.double("Hello")
"HelloHello"
```

---

## Conditionals

#### Multi-clause

```elixir
defmodule TestList do
  def empty?([]), do: true
  def empty?([_|_]), do: false
end

iex(37)> TestList.empty? []
true
iex(38)> TestList.empty? [1,2,3]
false
```

```elixir
defmodule LineCounter do
  def count(path) do
    File.read(path)  # File.read/1 returns either {:ok, contents} or {:error, reason}
    |> line_number
  end

  # The success branch.
  defp line_number({:ok, contents}) do
    contents
    |> String.split("\n")
    |> length
  end

  # The error branch.
  # Alternatively, we could omit the error branch, then when File.read/1 raises
  # an error, it will be propagated back to the caller.
  defp line_number(error), do: error
end
```

#### The `if` macro

```elixir
iex(60)> if true, do: "It's true", else: "It's false"
"It's true"
iex(61)> if false, do: "It's true", else: "It's false"
"It's false"
```

#### The `cond` macro
- Raises an error if none of the conditions is satisfied

```elixir
def max(a, b) do
  cond do
    a >= b -> a
    true   -> b  # The default clause
  end
end
```

#### The `case` macro
- Does the same thing as multi-clause functions do

```elixir
def max(a,b) do
  case a >= b do
    true -> a
    false -> b
  end
end
```

---

## Recursion for implementing loops
- Recursion is used in place of traditional loop constructs because looping does
not work in functional programming where data is immutable
- Recursive function calls can potentially consume the entire memory
- Erlang optimizes recursive function calls when the function is the last evaluated
expression (tail-call optimization)

```elixir
defmodule Fact do
  def fact(0), do: 1
  def fact(n), do: n * fact(n - 1)

  def print_range(a, b) do
    a..b |> Stream.map(&Fact.fact/1) |>Enum.each(&IO.puts/1)
  end
end

iex(54)> Fact.print_range(1, 5)
1
2
6
24
120
:ok
```

#### Tail vs non-tail recursion
- Usually non-tail recursion is more elegant
- Use tail recursion for large number of iterations

```elixir
# Non-tail recursion
defmodule ListHelper do
  def sum([]),            do: 0
  def sum([head | tail]), do: head + sum(tail)
end

iex(48)> ListHelper.sum([])
0
iex(49)> ListHelper.sum([1])
1
iex(50)> ListHelper.sum([1, 2])
3
```

```elixir
# Tail recursion (tail-call optimization)
defmodule ListHelper do
  def sum(list), do: do_sum(0, list)

  defp do_sum(current_sum, []),            do: current_sum
  defp do_sum(current_sum, [head | tail]), do: do_sum(current_sum + head, tail)
end
```

---

## Higher-order functions
- a function that takes function(s) as its input and/or function(s)

```elixir
iex(8)> Enum.map(
          1..3,
          fn(x) -> 2 * x end
        )
[2, 4, 6]

Enum.map(1..3, &(2 * &1))  # Shorthand
```

```elixir
iex(11)> Enum.filter(
           1..3,
           fn(x) -> rem(x, 2) == 1 end
         )
[1, 3]

Enum.filter(1..3, &(rem(&1, 2) == 1))  # Shorthand
```

```elixir
iex(17)> Enum.reduce(
           1..3,
           0,
           fn(x, acc) -> acc + x end
         )
6

Enum.reduce(1..3, 0, &(&1 + &2))  # Shorthand
Enum.reduce(1..3, 0, &+/2)        # Shorthand
```

```elixir
Enum.reduce(
  [1, "hello", 2, :world, 3],
  0,
  fn
    (element, sum) when is_number(element) ->
      sum + element
    (_, sum) -> sum
  end
)
```

---

## Comprehensions
- Powerful when performing nested iterations over multiple collections
- Can return anything that is collectable including lists, maps and file streams
- Often help us write more elegant code than Enum functions

```elixir
iex(23)> for x <- 1..3, do: 2 * x
[2, 4, 6]
```

```elixir
for x <- 1..9, y <- 1..9, do: IO.puts "#{x} * #{y} = #{x * y}"
```

```elixir
multiplication_table = for x <- 1..9, y <- 1..9,
                       into: %{},           # Specify the collectable
                       do: {{x, y}, x * y}  # Return a {factor, product} tuple

iex(34)> multiplication_table[{3, 4}]
12
```

```elixir
multiplication_table = for x <- 1..9, y <- 1..9,
                       x <= y,              # Comprehension filter
                       into: %{},           # Specify the collectable
                       do: {{x, y}, x * y}  # Return a {factor, product} tuple

iex(36)>  multiplication_table[{3, 4}]
12
iex(37)>  multiplication_table[{4, 3}]
nil
```

---

## Streams
- Lazy enumerables
- Useful when we need to compose multiple transformations of the same list
- Useful for slow and potentially large enumerable input (e.g., parsing a file)
- Describe the computation instead of iterating immediately

```elixir
iex(44)> Enum.map(1..3, &(2 * &1))
[2, 4, 6]

iex(45)> stream = Stream.map(1..3, &(2 * &1))
#Stream<[enum: 1..3, funs: [#Function<46.87278901/1 in Stream.map/2>]]>
iex(46)> stream |> Enum.take(1)
[2]
```

```elixir
[3, -12, "hoge", 29, -23, 37]              |>
Stream.filter(&(is_number(&1) and &1 > 0)) |>
Stream.map(&{&1, :math.sqrt(&1)})          |>
Stream.with_index                          |>
Enum.each(
  fn({ {input, result}, index }) ->
    IO.puts "#{index + 1}. sqrt(#{input}) = #{result}"
  end
)
# 1. sqrt(3) = 1.7320508075688772
# 2. sqrt(29) = 5.385164807134504
# 3. sqrt(37) = 6.082762530298219
# :ok
```
