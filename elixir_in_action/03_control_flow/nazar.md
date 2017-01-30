# Chapter 3. Control Flow

> ... and the resulting code is no more complicated than a typical OO solution.

I'd hope to think that "complexity" is subjective in this case.

> The result of a match expression is always the right-side term you match.

> When matching a map, the left-side pattern doesn't need to contain all the keys from right-side term.

String matching was pretty handy:

```elixir
iex(2)> url = "https://somedomain.com/some_entity/12345"
"https://somedomain.com/some_entity/12345"
iex(3)> "https://somedomain.com/some_entity/" <> entity_id = url
"https://somedomain.com/some_entity/12345"
iex(4)> entity_id
"12345"
```
In fact, where was this knowledge when I needed it when I was using Regular Expressions for something like that T_T

> Match expressions can be chained

> You can provide multiple definitions of the same function with the same arity.

Confusing at first, powerful later, especially handling tuples.

> If an error is raised from inside the guard, it won't be propagated, and the guard expression will return false.

Guards help implement polymorphic functions where depending on the type of input you get output that handles that input correctly.

```elixir
iex(5)> defmodule Naz do
...(5)>   def add(x, y) when is_number(x) and is_number(y), do: x + y
...(5)>   def add(x, y) when is_binary(x) and is_binary(y), do: x <> y
...(5)> end
iex(6)> Naz.add 1, 2
3  
iex(7)> Naz.add "test ", "strings"
"test strings"
```

Looping is implemented via multiclause functions. I still find that odd vs ML style way of doing it (with case, in Elixir's case).

```elixir
iex(8)> if true, do: true, else: false
true
iex(9)> x = 1
1
iex(10)> if x > 1, do: :greater
nil
iex(11)>
```

Nil for an unmet condition will SO throw me off :(

```elixir
iex(12)> defmodule Naz do
...(12)>   def sum([]), do: 0
...(12)>   def sum([hd | tl]), do: hd + sum(tl)
...(12)> end
iex(13)> Naz.sum [1,2,3]
6
```

vs

```elixir
iex(14)> defmodule Naz do
...(14)>   def sum(list) do
...(14)>     case list do
...(14)>       [] -> 0
...(14)>       [hd | tl] -> hd + sum tl
...(14)>     end
...(14)>   end
...(14)> end

iex(15)> Naz.sum [3,2,1]
6
```

> If the last thing the function does is calls itself or another function, you are dealing with tail call which is optimized (via tail call optimization)

> Tail recursive function can run forever without consuming additional memory

The reason the above function is not tail recursive is due to the `hd + sum tl` where `+` is the last call.

Rewriting needs to ensure the function itself is called. An accumulator can be used to keep track of the sum.

```elixir
iex(17)> defmodule Naz do              
...(17)>   def sum(l), do: do_sum(0, l)         
...(17)>   defp do_sum(s, []), do: s             
...(17)>   defp do_sum(s, [h|t]), do: do_sum(s + h, t)  
...(17)> end

iex(18)> Naz.sum [4,3,2,1]
10
```

Tail recursive version should be reserved for arbitrarily long lists. Non TR approach is often more elegant and more performant.

`Enum.reduce/3`
> The lambda receives the current accumulator value and the element from the enumerable.

Can be shorthanded (justifiably?) using &/ notation:

```elixir
iex(19)> Enum.reduce([1,2,3], 0, &+/2)
6
```

Compression syntax:
```elixir
iex(20)> for x <- [1,2,3,4], do: x * x
[1, 4, 9, 16]
```

Streams are a special kind of enumerables that can be useful for doing lazy composable operations over anything enumerable.

```elixir
iex(22)> stream = [1,2,3] |> Stream.map(fn x -> x * x end)
#Stream<[enum: [1, 2, 3], funs: [#Function<47.36862645/1 in Stream.map/2>]]>
iex(23)> Enum.to_list(stream)
[1, 4, 9]
```

Useful for slow and large enumerable input (like files, etc)
