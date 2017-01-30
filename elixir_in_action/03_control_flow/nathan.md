# 3.x Control Flow

## 3.1

* allows constant on left side
* {:ok, data} or {:error, reason} commonly used
* `_` or `_{varname}` for things you want to ignore
* `^` on left side doesn't allow variable binding
* same var used twice on left side must be the same "thing"
* `[ first | rest ]` common pattern for recursively matching on lists
* partial-mapping on Maps
* binary matching is extremely useful when needed.
* Strings can be broken up by bits, graphemes, and/or codepoints
* Match chaining is used commonly in params ```def([frst | rest] = list) do```

## 3.2

* Multiclause functions must have the same arity
* Multiclause functions are ordered from top to bottom
* Guards have a limited set of allowed operators/functions
  * great read on why http://keathley.io/2016/04/09/elixir-guard-clauses.html
* you can only use `and`, `or`, `not` and not `&&`, `||`, `!` in guard clauses
* errors in guard clauses are captured and assumed to be a mismatch thus continuing through multiclause functions
* I like the lambda multiclause over the def multiclause as they seem better grouped together
  * dug into why no `defmulti` option. one coversation on Elixir Core. may suggest again.

## 3.3
* Love the refactoring of `File.read` to remove conditions and use pipes
* Seems cond would be needed for any `if` that needs an `elsif` type of clause

## 3.4
* Get comfortable with recursion if you aren't already
* Go for readability. tail recursion is not required in most calculations. it is required for `receive` and other never ending recursions
* not a fan of list comprehensions. prefer Enum Module options
* Streams are awesome and I should find more ways to use them regularly

## Exercises

``` elixir
defmodule FileLineData do

  def line_lengths!(path) do
    File.stream!(path)
    |> Stream.map(&(String.replace(&1, "\n", "")))
    |> Enum.map(&(String.length(&1)))
  end

  def longest_line_length!(path) do
    File.stream!(path)
    |> Stream.map(&(String.replace(&1, "\n", "")))
    |> Enum.reduce(0, &(if String.length(&1) > &2, do: String.length(&1), else: &2))
  end

  def longest_line!(path) do
    File.stream!(path)
    |> Stream.map(&(String.replace(&1, "\n", "")))
    |> Enum.reduce("", &(if(String.length(&1) > String.length(&2), do: &1, else: &2)))
  end

  def words_per_line!(path) do
    File.stream!(path)
    |> Stream.map(&(String.replace(&1, "\n", "")))
    |> Enum.map(&(length(String.split(&1))))
  end

end

```
