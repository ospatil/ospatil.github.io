---
title: Initial steps towards intuition into recursion in FP
layout: post
---
I'm learning functional programming paradigm in general and Elixir in particular. It's quite a different way of doing things with pure functions, recursion, immutable data structures and absence of the usual constructs such as loops one is so used to. I have been reading quite a bit and practising small exercises with recently started activity of solving the [99 problems](https://ocaml.org/learn/tutorials/99problems.html). But so far, the intuition into using recursion instead of loops to work with lists seemed to elude me. All these references to breaking down a problem into smaller ones that repeat and converge towards _base case_ seemed easier said than done. With all that at the back of mind, I went to sleep day before yesterday thinking about _problem 7 - Flatten a nested list structure_ and woke up with a light-bulb moment :smiley:. While it might be a trivial problem for pundits, I am summing it all up here step by step in a hope that complete noobs like myself may pick up a thing or two from it.

## The problem

> Flatten a nested list structure.

## Solution

You can run find the complete code example here -  [https://gist.github.com/ospatil/744611085cd870787fe2e1188e6212cf](https://gist.github.com/ospatil/744611085cd870787fe2e1188e6212cf). You can execute it with the command `elixir p7.exs` and trace the steps mentioned below by looking at the revisions.

### Step 1

Let's begin with solution for the simplest possible case - an empty list. Flattening an empty list will return an empty list.

```elixir
defmodule P7 do
  # public function that calls a private one with accumulator
  def flatten(list), do: do_flatten(list, []) # 1
  # base case - empty list
  defp do_flatten([], result), do: result # 2
end

# Test case code omitted for brevity. You can find it in gist.
```

1. This is a public function. When users calls it, we invoke a private function `do_flatten` with a list as an accumulator that will collect the flattened elements to be returned to the user.
2. This is the base case for recursion. When invoked with empty list, it just returns the `results` accumulator that holds the flattened elements.

We execute the file, test case passes and we are good.

### Step 2

Now let's consider the next case: a list with one element - either a list or simple value.

```elixir
defmodule P7 do
  # code from step 1 omitted for brevity

  # next two clauses are for lists with only one element - either a list or value
  defp do_flatten([head], result) when is_list(head), do: do_flatten(head, result) # 1
  defp do_flatten([head], result), do: [head | result] # 2
end

# Test case code omitted for brevity. You can find it in gist.
```

1. This clause is invoked when the list element is list. The `when is_list(head)` constraint ensures that. All we do in the implementation is recursively call with the value inside the list and accumulator: if the value is list, the same clause is invoked; if a simple element,  clause 2 will be invoked.
2. This clause in invoked when list element is a simple value. We need to add the element to the `result` accumulator. You'll notice that we are prepending to the `result` list instead of appending it. Why is that? Won't the final result be in reverse order? Absolutely, that's right. but I learnt that in functional languages list _prepend_ is a constant time operation while _append_ is linear. Therefore, prepending instead of appending and then reversing in one go is a fairly common approach and that's what we'll use.

We execute the file, the test case passes and we are ready to proceed further.

### Step 3

Now let's consider list with multiple elements.

```elixir
defmodule P7 do
  # public function that calls a private one with accumulator
  def flatten(list), do: Enum.reverse(do_flatten(list, [])) # 3
  # base case - empty list
  defp do_flatten([], result), do: result
  # next two clauses are for lists with only one element - either a list or value
  defp do_flatten([head], result) when is_list(head), do: do_flatten(head, result)
  defp do_flatten([head], result), do: [head | result]
  # next two clauses for list with many elements
  defp do_flatten([head | tail], result) when is_list(head), do: do_flatten(tail, do_flatten(head, result)) # 1
  defp do_flatten([head | tail], result), do: do_flatten(tail, [head | result]) # 2
end

# Test case code omitted for brevity. You can find it in gist.
```

You'll notice the code added for this step is very similar to step 2. I added two clauses - one for when head of list to be flattened (i.e. the current element under consideration) is a list and other for simple element.

1. This is invoked when the head is list. The implementation is quite important. We have head that is a list and tail is a list too. So we need to flatten head first and then continue flattening the tail. We achieve this by `do_flatten(tail, do_flatten(head, result))` where we call `do_flatten` on head passing the origin `result` accumulator and use the list it returns (which contains tha flattened elements) as accumulator for flattening the tail.
2. This is invoked when the head is a simple value. We simply append the head to the accumulator and continue flattening tail.
3. As mentioned in step 2, the accumulator holds elements in reverse order due to prepending. We reverse it using `Enum.reverse` before returning to the user.

We execute the file and the test cases pass. Yay!

### Step 4

We already have something like full implementation that's able to flatten arbitrary depth lists, so what's left?
Just a little cleanup, that's it. You will notice that with the clauses in step 3 in place, the clauses we added in step 2 for dealing with single element list are not needed. Here is the final implementation after removing those -

```elixir
defmodule P7 do
  # public function that calls a private one with accumulator
  def flatten(list), do: Enum.reverse(do_flatten(list, []))
  # base case - empty list
  defp do_flatten([], result), do: result
  # next two clauses for list with many elements
  defp do_flatten([head | tail], result) when is_list(head), do: do_flatten(tail, do_flatten(head, result))
  defp do_flatten([head | tail], result), do: do_flatten(tail, [head | result])
end

# Test case code omitted for brevity. You can find it in gist.
```
