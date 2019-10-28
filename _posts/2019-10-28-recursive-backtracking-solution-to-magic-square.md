---
layout: post
current: post
cover: assets/images/posts/recursive-backtracking.jpg
navigation: True
title: A backtracking solution to the Magic Square problem
date: 2019-10-27 20:00:00
tags: [ruby]
category: [ruby]
class: post-template
subclass: 'post'
author: manish
---

Recursion can be daunting to many developers, novice and experienced alike. But
certain problems can be solved quite easily using recursive algorithms.

One such algorithm is recursive backtracking.

Let's understand what the `Magic square` problem is, and device a solution using
backtracking in **Ruby**.

### Problem

A magic square is an NxN matrix of distinct integers ranging from 1 to 
N^2, arranged such that the individual sum of all rows, all columns, and both
diagonals comes out to be the same constant.

Example:

{% highlight ruby %}
matrix = [
  [ 1, 15, 14,  4],
  [10, 11,  8,  5],
  [ 7,  6,  9, 12],
  [16,  2,  3, 13]
]
{% endhighlight %}

The sum of rows, columns and both the diagonals comes out to be 34.

[You can read more about the problem here.](https://www.geeksforgeeks.org/magic-square/)

### Approach

The approach, on which we are going to build our solution is `backtracking`.

First off, what is backtracking?

Backtracking is a recursive algorithmic technique for solving problems 
incrementally, one step at a time, while removing those solutions that fail to 
satisfy the constraints of the problem.

[More on the backtracking here.](https://www.geeksforgeeks.org/backtracking-introduction/)

### Solution

Let's take an example to find the solution to.

{% highlight ruby %}
matrix = [
    [ 1, 0, 0,  4],
    [10, 0, 8,  5],
    [ 0, 0, 9,  0],
    [16, 0, 0, 13],
  ]
{% endhighlight %}

Here `0` represents the missing numbers we need to find.

Now, for every recursive algorithm, we need to have a base case. That is, we need to
let the algorithm know when to stop, else it will go in an infinite loop.

For our case the base case is pretty simple. To check whether the given matrix 
for each iteration is solved or not. And as mentioned earlier, a matrix is solved
when the individual sum of its rows, columns, and both diagonals is constant.
For our 4x4 example, that constant is 34.

So, let's write some methods to check if a matrix is solved or not.

The `row_solved?` method reduces the given row to the sum of its elements, and compares
it with our constant, i.e. 34.

The `rows_solved?` method iterates over each row, and returns `false` if any row's sum
doesn't match our constant, returns `true` otherwise.

The `columns_solved?` is calls the `rows_solved?` method with a transposed matrix
as its input, which in-turn will iterate over columns rather than rows.

{% highlight ruby %}
def row_solved?(row)
  row.reduce(0, :+) == 34
end

def rows_solved?(matrix)
  matrix.each do |row|
    return false unless row_solved? row
  end

  true
end

def columns_solved?(matrix)
  rows_solved?(matrix.transpose)
end
{% endhighlight %}

Similarly, `diagonals_solved?` method collects and checks whether both diagonals are solved
using our `row_solved?` method.

{% highlight ruby %}
def diagonals_solved?(matrix)
  diagonal_1 = (0..3).collect { |i| matrix[i][i] }
  diagonal_2 = (0..3).collect { |i| matrix[i][3 - i] }

  row_solved?(diagonal_1) && row_solved?(diagonal_2)
end
{% endhighlight %}

Here, we need one more check. That is, to check whether all the elements in the
matrix are filled or not.
That is basically to check whether our matrix contains any `0`

{% highlight ruby %}
def has_null?(matrix)
  matrix.flatten.select(&:zero?).any?
end
{% endhighlight %}

Now that all our conditions are ready, our main check method `is_solved?` looks like this.

{% highlight ruby %}
def is_solved?(matrix)
  !has_null?(matrix) &&
    rows_solved?(matrix) &&
    columns_solved?(matrix) &&
    diagonals_solved?(matrix)
end
{% endhighlight %}

With our base case ready, we can move forward with the next step, which is to fill
the remaining numbers in our matrix.

Here, we'll go a bit deep in the recursive backtracking realm. It might be a little
confusing, but you'll get a hang of it.

We will start filling our matrix one by one with the remaining numbers, and on every step, we check
if we have reached a valid solution. If we have not reached a valid solution, we 
might have put something wrong on the way, so we go a step back, fill a different number,
and move ahead with our solution, and we repeat this cycle of filling, checking and backtracking until we get a valid solution
or until we have tried all combinations, then there is no solution.

Let's call our recursive method `solve`. It takes a matrix as its argument. 
It returns `true` if the given matrix is solved, and `false` if it cannot be solved.

{% highlight ruby %}
def solve(matrix)
  return true if is_solved? matrix

  remaining(matrix).each do |remaining_number|
    slot = find_empty_slot matrix

    matrix[slot[:i]][slot[:j]] = remaining_number

    return true if solve matrix

    matrix[slot[:i]][slot[:j]] = 0
  end

  false
end
{% endhighlight %}

We start with our base case, and check whether the given matrix is solved. If so, then we return `true`.
If not, then we'll iterate over the remaining numbers in the matrix.

{% highlight ruby %}
def remaining(matrix)
  [*1..16] - matrix.flatten.reject(&:zero?)
end
{% endhighlight %}

Once in the loop, we'll first search an empty slot to fill in a number into.

{% highlight ruby %}
def find_empty_slot(matrix)
  matrix.each_with_index do |row, i|
    row.each_with_index do |ele, j|
      return { i: i, j: j } if ele.zero?
    end
  end
end
{% endhighlight %}

Now, we fill in the number into the slot, and call our `solve` method again.
What changed here is that we have added another number to our matrix, whether it 
is correct or not, is a question for later, but with this step we have incremented a step 
further.

Now, that we have called the `solve` method, it again checks if the given matrix
is solved or not. If yes, it will return true. If not, it'll calculate the remaining
elements in the matrix, which if you remember, has an extra number, we put in the previous step. So,
now we have less numbers to fill.

Now imagine if we filled our last number, our matrix looks something like this now.
{% highlight ruby %}
matrix = [
    [ 1,  2,  3,  4],
    [10,  6,  8,  5],
    [ 7, 11,  9, 12],
    [16, 14, 15, 13]
  ]
{% endhighlight %}

The last number we filled is 15. So now the code checks whether it is solved or not.
It is not. So, we might have put something wrong on the way. So, we take a step back
which is 14, which we filled wrong, and try to fill other possible numbers there, in this case
the only other number is 15. So we fill 15 first, and then 14.

Now we have:

{% highlight ruby %}
matrix = [
    [ 1,  2,  3,  4],
    [10,  6,  8,  5],
    [ 7, 11,  9, 12],
    [16, 15, 14, 13]
  ]
{% endhighlight %}

We again check if we have solved it. It's not solved. So, again we backtrack a step
and try further combinations. So, we move back to 12, replace it with the number next
in line i.e. 14 and move further. And we get:

{% highlight ruby %}
matrix = [
    [ 1,  2,  3,  4],
    [10,  6,  8,  5],
    [ 7, 11,  9, 14],
    [16, 12, 15, 13]
  ]
{% endhighlight %}

This process continues till we reach the final solution which is:

{% highlight ruby %}
matrix = [
  [ 1, 15, 14,  4],
  [10, 11,  8,  5],
  [ 7,  6,  9, 12],
  [16,  2,  3, 13]
]
{% endhighlight %}

Here is a look at the `solve` method again.

{% highlight ruby %}
def solve(matrix)
  # Base case
  return true if is_solved? matrix

  # Iterating over remaining numbers
  remaining(matrix).each do |remaining_number|
    # Finding a slot to fit in a number
    slot = find_empty_slot matrix

    # Filling the number in the found slot
    matrix[slot[:i]][slot[:j]] = remaining_number

    # Recursive call to the method
    # to check if its solved or not
    return true if solve matrix

    # We reach here when we didn't find a solution
    # in the above iteration, so we reset the slot
    # to zero, and go ahead with our loop
    matrix[slot[:i]][slot[:j]] = 0
  end

  false
end
{% endhighlight %}

### Conclusion

The beauty of recursion is that you don't need to care about the intermediate conditions.
You just need to identify, and handle the base case effectively. And that's it,
recursion will do the rest for you.

[Here's](https://github.com/nikamanish/codebase/blob/master/backtracking/magic_square.rb) the link to the full code, if you'd like to experiment with it by yourself. Be vary of the base case 
though, it may cause some nasty infinite loops, which are hard to debug. I have also added
a `print_arr` method, if you'd like to see what's going on underneath.

