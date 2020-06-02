# Elixir tips

## Spliting strings with sigils

```elixir
iex> text = "foo bar"
iex> ~w/#{text}/
["foo", "bar"]
```

## Creating an array containing the alphabet

A char list containing the alphabet can be obtained by doing the following:

```elixir
iex> for n <- ?a..?z, do: n
'abcdefghijklmnopqrstuvwxyz'
```

If desired, we can pass each bitcode to a binary to represent each letter as a string:

```elixir
iex> for n <- ?a..?z, do: << n :: utf8 >>
["a", "b", "c", "d", "e", "f", "g", "h", "i", "j", "k", "l", "m", "n", "o", "p","q", "r", "s", "t", "u", "v", "w", "x", "y", "z"]
```

## Multiple expressions on single line

```elixir
x=1; y=2; z=3; a=4; b=5; c=6; d=7; e=8; f=9;
```

## Finding list of bound values in iex

```elixir
iex> binding
[]
iex> name = "blackode"
"blackode"
iex> blog = "medium"
"medium"
iex> binding
[blog: "medium", name: "blackode"]
```

## Integer to float

```elixir
iex> 5/1
5.0
iex> 5+0.0
5.0
```

## If/else alternative

```elixir
iex(1)> x = 7
7
iex(2)> y=10
10
iex(3)> greatest = x>y && x || y
10
iex(4)> smallest = x<y && x || y
7
```

## Multiple OR

This is not the recommended approach because in regular approach when the condition evaluates to true , it stops executing the remaining conditions which saves time of evaluation unlike this approach which evaluates all conditions first in list. This is just bad but good for discoveries.

### Regular Approach

```elixir
find = fn(x) when x>10 or x<5 or x==7 -> x end
```

### Our Hack

```elixir
hell = fn(x) when true in [x>10, x<5, x==7] -> x end
```

## Inspecting data types

### `i/1`

Prints information about the data type of any given term.

```elixir
iex> i Hello

Term
  Hello
Data type
  Atom
Raw representation
  :"Elixir.Hello"
Reference modules
  Atom
Implemented protocols
  IEx.Info, Inspect, List.Chars, String.Chars
```

## Creating custom sigils

```elixir
defmodule MySigils do
  # returns the downcasing string if option l
  # is given then returns the list of downcase letters

  def sigil_l(string, []), do: String.downcase(string)
  def sigil_l(string, [?l]) do
    string
    |> String.downcase()
    |> String.graphemes()
  end

  # returns the upcasing string if option l
  # is given then returns the list of downcase letters

  def sigil_u(string, []), do: String.upcase(string)
  def sigil_u(string, [?l]) do
    string
    |> String.upcase()
    |> String.graphemes()
  end
end
```

usage:

```elixir
iex> import MySigils
iex> ~l/HELLO/
"hello"
iex> ~l/HELLO/l
["h", "e", "l", "l", "o"]
iex> ~u/hello/
"HELLO"
iex> ~u/hello/l
["H", "E", "L", "L", "O"]
```

## Custom errors

```elixir
# bug_error.ex
defmodule BugError do
   defexception message: "BUG BUG .." # message is the default
end
```

usage:

```elixir
$ iex bug_error.ex

iex> raise BugError
** (BugError) BUG BUG ..

iex> raise BugError, message: "I am Bug.." #here passing the message dynamic
** (BugError) I am Bug..
```

## Get a value from nested maps

```elixir
iex> nested_map = %{name: %{first_name: "blackode"}}

# Retrieving the Key:
iex> get_in(nested_map, [:name, :first_name])
"blackode"

iex> get_in(nested, [:name, :last_name])
nil
# returns nil when key is not present
```

## `with` statement benefits

The special form with is used to chain a sequence of matches in order and finally return the result of do: if all the clauses match. However, if one of the clauses does not match, its result of the miss matched expression is immediately returned.

```elixir
iex> with 1 <- 1+0,
...>      2 <- 1+1,
...>      do: IO.puts "all matched"
"all matched"

iex> with 1 <- 1+0,
...>      2 <- 3+1,
...>      do: IO.puts "all matched"
4

# since 2 <- 3+1 is not matched so the result of 3+1 is returned
```

## Using module `Kernel` with pipelines

```elixir
:input
|> do_something
|> do_another_thing
|> Kernel.||(:default_output)       # <-- This line
|> do_something_else
```

## Short circuit operators `&&` and `||`

### `||` operator

These replaces the nested complicated conditions. These are my best friends in the situations dealing with more complex comparisons.

The `||` operator always returns the first expression which is true. Elixir doesn’t care about the remaining expressions, and won’t evaluate them after a match has been found.

```elixir
iex> false || nil || :blackode || :elixir || :jose
:blackode
```

Here if you observe the first expression is false next nil is also false in elixir next `:blackode` which evaluates to true and its value is returned immediately with out evaluating the `:elixir` and `:jose` . Similarly if all the statements evaluates to false the last expression is returned.

### `&&` operator

```elixir
iex> true && :true && :elixir && 5
5
iex> nil && 100
nil
iex> salary = is_login && is_admin && is_staff && 100_000
```

This `&&` returns the second expression if the first expression is true or else it returns the first expression with out evaluating the second expression. In the above examples the last one is the situation where we encounter to use the `&&` operator.

## Finding substrings

```elixir
iex> "blackode" =~ "kode"
true
iex> "blackode" =~ "medium"
false
iex> "blackode" =~ ""
true
```

## Check if a module is loaded or not

```elixir
iex> Code.ensure_loaded?(:kernel)
true
iex> Code.ensure_loaded(:kernel)
{:module, :kernel}
```

Similarly we are having ensure_compile to check whether the module is compiled or not…

## Binary to capital atom

```elixir
iex> String.to_atom("Blackode")
:Blackode
iex> Module.concat(Elixir,"Blackode")
Blackode
```

## Pattern match VS destructure

We all know that `=` does the pattern match for left and right side. We cannot do `[a, b, c] = [1, 2, 3, 4]` this raise a `MatchError`

```elixir
iex> [a, b, c] = [1, 2, 3, 4]
 ** (MatchError) no match of right hand side value: [1, 2, 3, 4]
```

We can use destructure/2 to do the job.

```elixir
iex> destructure [a, b, c], [1, 2, 3, 4]
[1, 2, 3]
iex> {a, b, c}
{1, 2, 3}
```

If the left side is having more entries than in right side, it assigns the nil value for remaining entries..

```elixir
iex> destructure([a, b, c], [1])
[1, nil, nil]
iex> {a, b, c}
{1, nil, nil}
```

## Inspect with :label

```elixir
iex> IO.inspect [1, 2, 3], label: "the list"
the list: [1, 2, 3]
[1, 2, 3]
```

## Subtraction over lists

### on lists

```elixir
iex> [1, 2, 3, 4] -- [1, 2]
[3, 4]
iex> [1, 2, 3, 4, 1] -- [1]
[2, 3, 4, 1]
iex> [1, 2, 3, 4, 1] -- [1, 1]
[2, 3, 4]
iex> [1, 2, 3, 4] -- [6]
[1, 2, 3, 4]
```

### on charlists

```elixir
iex> 'blackode' -- 'ode'
'black'
iex> 'blackode' -- 'z'
'blackode'
iex> 'blackode' -- 'hello'
'backd'
```

## Aliasing multiple modules

```elixir
alias Hello.{One,Two,Three}
# The above line is same as the following:
alias Hello.One
alias Hello.Two
alias Hello.Three
```

## Substring

```elixir
iex> String.slice("blackode", 1..-1)
"lackode"
iex> String.slice("blackode", 0..-4)
"black"
```

## Initializing multiple with same value

```elixir
iex> x = y = z = 5
5
iex> {x, y, z}
{5, 5, 5}
```

## Checking the closeness of strings

```elixir
iex> String.jaro_distance("ping", "pong")
0.8333333333333334
iex> String.jaro_distance("color", "colour")
0.9444444444444445
iex> String.jaro_distance("foo", "foo")
1.0

# This returns a float value in the range 0..1
```

## Chain of OR in guards

```elixir
def print_me(thing)
  when is_integer(thing)
  when is_float(thing)
  when is_nil(thing) do
    "I am a number"
end
```

## Replacing a key in a map

```elixir
details = %{name: "john", address1: "heaven", id1: 42}
%{address1: "heaven", id1: 42, name: "john"}

iex> Map.new(details, fn
  {:address1, address} -> {:address, address}
  {:id1, id} -> {:phone, id}
  x -> x
end)
# output
%{address: "heaven", name: "john", phone: 42}
```

## Generate 4 digit random number


```elixir
# 0001..9999

"~4..0B"
|> :io_lib.format([:rand.uniform(10_000) - 1])
|> List.to_string()
```
