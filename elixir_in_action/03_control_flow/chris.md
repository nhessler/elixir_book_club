# Control Flow

"Classical conditional constructs such as `if` and `case` are often
replaced with _multiclause functions_."

## TLDR

- Pattern matching evaluates an expression and binds variables
- Function arguments are patterns, which can be used to create multiclause functions
- Multiclause functions are used to implement conditional branching
- Recursion is the primary method of iteration
- Higher-order functions make iteration easier
- Comprehensions can be used to transform, filter, and join enumerables
- Streams allow lazy composition of transformations

## The Details

### 3.1 Pattern matching

`=` is called the match operator. It's not assignment.

### 3.1.1 The match operator

- At runtime, the left side of the `=` operator is matched to the right side.
- `<pattern> = <expression>`

### 3.1.2 Matching tuples

Match operator can be used to destructure tuples:

```elixir
iex(1)> {_, {hour, minute, second}} = :calendar.local_time
{{2017, 2, 4}, {15, 15, 52}}
iex(2)> hour
15
```

If the match fails, a MatchError will be raised.

```elixir
iex(3)> {_, {hour, minute, second}} = "lolnotgonnawork"
** (MatchError) no match of right hand side value: "lolnotgonnawork"
```

### 3.1.3 Matching constants

- You can put constants on the left of `=` (showing that it is not assignment).

```elixir
iex(4)> 1 = 1
1
```

- That's not very useful by itself, but a common idiom is returning
`{:ok, result}` or `{:error, reason}`.
- This makes it easy to match on success and error conditions.

### 3.1.4 Variables in patterns

- `_` is the anonymous variable and matches anything
- patterns can be arbitrarily nested
- `^` is the pin operator, which allows you match against the contents
  of the variable

### 3.1.5 Matching lists

- Matching lists is usually done with recursion
- Each non-empty list is a recursive list of `[head | tail]`

### 3.1.6 Matching maps

- When matching a map, the left-side pattern doesn't need to contain all
  the keys from the right-side term

### 3.1.7 Matching bitstrings and binaries

- Matching bitstrings and binaries can be useful when parsing packed binary
  content that comes from a file, an external device, or network
- You can parse a string using matching:

```elixir
iex(5)> "ping " <> url =  "ping www.example.com"
"ping www.example.com"
iex(6)> url
"www.example.com"
```

### 3.1.8 Compound matches

- Match operator is right-associative
- Matches can be chained:

```elixir
iex(7)> {_, {hour, minute, seconds}} = ({date, time} = :calendar.local_time)  
{{2017, 2, 4}, {15, 43, 36}}
iex(8)> time                                                                
{15, 43, 36}
iex(9)> hour
15
```

### 3.1.9 General behavior

Pattern matching does two things:
- Asserts expectations about the right-side term
- Bind some parts of the the term variables from the pattern

### 3.2 Matching with functions

Pattern-matching function arguments underpins one of the most important
features of Elixir: _multiclause functions_.

### 3.2.1 Multiclause functions

- Function overloading is performed by specifying multiple clauses
- From the caller's perspective, a multiclause function is a single function
- The runtime tries to select the clauses using the order in the source code

> **CONVENTION**: Group clauses of the same function together.

### 3.2.2 Guards

- A guard can be specified by using the `when` clause after the argument list
- Elixir terms can be compared even if they're not the same type

> **TYPE ORDERING**: number < atom < reference < fun < port < pid < tuple < map < list < bitstring (binary)

- The set of operators and functions that can be called from guards is limited

### 3.2.3 Multiclause lambdas

Multiclause lambdas can be useful with higher-order functions.

```elixir
iex(10)> test_num = fn
...(10)>   x when is_number(x) and x < 0 -> :negative
...(10)>   0 -> :zero
...(10)>   x when is_number(x) and x > 0 -> :positive
...(10)> end
#Function<6.52032458/1 in :erl_eval.expr/5>
iex(11)> test_num.(-1)
:negative
iex(12)> test_num.(0) 
:zero
iex(13)> test_num.(1)
:positive
```

### 3.3 Conditionals

### 3.3.1 Branching with multiclause functions

- Multiclause-powered recursion is used as a building block for looping
- Multiclause functions can clean up "main" code by removing conditionals
- There is nothing you can do with multiclauses that can't be done using
  traditional branching constructs, but it helps code become more declarative

### 3.3.2 Classical branching constructs

`if` and `unless`

- If the condition isn't met and the `else` clause isn't specified,
  the return value is `nil`

`cond` macro

- The result of `cond` is the result of the corresponding executed block
- If none of the conditions are satisfied, `cond` raises an error
- The `true` pattern ensures that the condition will always be satisfied

`case`

- evaluates an expression and matches it with a pattern clause
- If no clause matches, an error is raised
- Anonymous variable can be used to match any pattern: `_ -> ...`
- Seems to implement multiclause functions in one construct

### 3.4 Loops and iterations

- Principle looping tool is recursion

### 3.4.1 Iterating with recursion

### 3.4.2 Tail function calls

- Elixir (well, the Erlang VM) performs tail-call optimization 

### 3.4.3 Higher-order functions

 - Enum.each/2
 - Enum.map/2
 - Enum.filter/2
 - Enum.reduce/3 (may know this as inject or fold)

 > **NOTE**: Avoid writing elaborate lambdas. If there is too much logic,
   that's a sign that you may want to create a distinct function.

### 3.4.4 Comprehensions

- Comprehensions allow nested iterations over multiple collections
- Comprehensions can iterate through any enumerable
- Comprehensions can return anything that is collectable

### 3.4.5 Streams

- Streams are enumerables and useful for lazy operations (like generators)
- With a stream, we can apply filters and transformations to data, but
  we don't have to iterate over the collection multiple times
- Even if you stack multiple transformations on a stream, everything is
  performed on a single pass
- To make a lazy computation, return a lambda that performs the computation
- When the computation is materialized, the consumer can call the lambda
