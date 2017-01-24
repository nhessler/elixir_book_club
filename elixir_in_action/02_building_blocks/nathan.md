# Chapter Two

**h is so useful in iex!**
always forget it's there until I'm reminded. also means you have to support it by writing docs and examples.

*Note:* The `=` operator is not an assignment opperator. it's a match operator. it's the unification of each side that forces the binding of the unbound/unpinned variable.

*Oddity* Surprised to hear there is no actual module hierarchy. It's existence in Elixir is strictly a nicety. It makes sense thinking about, but it caught me off guard at first.

*Correction:* You can use multiline pipelines, but you must end each line with the `|>` instead of beginning each line.

**Example**

``` elixir
iex(1)> -5 |>
iex(1)> abs |>
iex(1)> Intetger.to_string |>
iex(1)> IO.puts
5
:ok
```
*Note:* `import` and `alias` do not have to be used at the module level. you can use them inside functions as well.

@moduledoc and @doc are great. and with ex_doc you have a powerful documentation system at your finger tips.

*Q???* Anyone use dializer or dialixir on a regular basis? if so, what would be your quick impression of their use?

*Oddity* No booleans, `true`, `false`, `nil` are really just `:true`, `:false`, `:nil`

*Q???* Aren't `HashDict` and `Dict` derpicated?

*Love:* I'm a big fan of the `&` operator. but it has it's limits. doesn't work well with too many arguments or arguments that need a name to give context.

*Note:* IOList is really useful for performance. still not sure when/where to use it, but should consider it as an alternative when using Linked Lists depending on use case.
