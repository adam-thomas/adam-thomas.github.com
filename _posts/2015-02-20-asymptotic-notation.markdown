---
layout: main
title: Asymptotic notation
---


# Asymptotic notation

**Asymptotic notation** is a useful way of expressing how quickly an algorithm will run on an input of any size.

You may have seen "Big-O notation" before:

```
O(n)
```

The idea of asymptotic notation is to concisely describe how quickly the code's execution time grows with its input size.  So-called "big-O" notation is the most commonly used form of asymptotic notation, and the one that's most likely to be important while developing software.

On small inputs, most programs will run comfortably, especially on modern computers.  On large inputs, however, their asymptotic performance becomes extremely important.  Imagine you have a list a thousand items long (far from implausible; some people have a thousand Facebook friends), and you have a task that involves operating on that list.

Solving that task will probably involve a loop.  Maybe it's a `for` loop, a list comprehension, even some kind of database operation.  One way or another, you have to iterate over it.  If your program just loops over the entire list once, then it'll run `n` iterations on a list of `n` items.  In our example, that means 1,000 loop iterations.  We can describe this program as being `O(n)`, or "order of n".

Now imagine you have a nested loop.  Which, itself, contains a loop.  For each iteration of the array, you're looping over the array, and in each of those iterations, you're doing another loop.  Who knows why - perhaps you're doing some kind of complex comparison.  Three nested loops of the same `n`-length array will take around `n` times `n` times `n` iterations, or `n^3`.  We can describe this program as being `O(n^3)`.

On a really small data set of only three or four items, the performance hit won't really be noticeable to us humans.  For the list in the example, the difference is remarkable.  The program with `O(n^3)` execution time will take around a million times as long to complete this task as the program with `O(n)` execution time!  That's the difference between five seconds and almost 58 days.


### Constructing Big-O statements

Once again, our big-O notation looks like this:

```
O(n)
```

`O()` is the most commonly-used bit of asymptotic notation, and is normally pronounced "order of".  It means that, above a certain size of input, your algorithm will take no more time than a static multiple of whatever's in the brackets.

That's a bit of a word salad, so let's break it down.

The expression inside the brackets - the `n` in `O(n)` - is written as a mathematical operation on at least one variable (n, here).  Any expression involving `n` and whichever other variables you choose to use is permitted.  Here are a few examples:

- `n + 2`
- `n^2 + 3n` - `n` squared, with three more `n`s on top
- `nx` - n times x, with x being another variable, such as the size of a different input data set

You can evaluate that expression for any value of n (and other variables), and get a number out of it.  For us, that number normally represents a number of loop iterations.

Let's call our expression `E` so we don't have to worry about what it is.  `E` is a mathematical operation on our variable, `n`.  It could be any of our three examples above, or anything else you fancy.  It could just be `n`.  We don't really mind.  We also have a constant value, `k`, which is just some arbitrary number.  We don't care what it is, either.

#### The all-important syntactic definition!

To say that a piece of code is `O(E)` means that:

- As long as `n` is at least a certain, constant size,
- the code takes no more than `kE(n)` iterations to run.

#### End all-important syntactic definition.

`kE(n)` is our constant number `k`, multiplied by the result of our operation being fed `n`, the input size.  We don't actually care what `k` is.  Its existence merely makes things more convenient for us when writing the big-O notation, since we can leave it off and not have to consider `O(n)`, `O(2n)`, `O(3n)` and so on differently.  They're all just `O(n)`.

The "as long as n is at least a certain size" bit is important.  Most sections of code will have at least some overhead that doesn't scale with the size of the input - housekeeping before and after the loop, initialising data structures, whatever.  This rule allows us to handwave that away and focus on the important characteristics of the program.  As a result, we can ignore anything other than the biggest `n` term in our expression.  This will usually be the one with the highest exponent.

Both of these neat shortcuts, taken together, mean that `O()` expressions simplify down to a very minimal form:

- `O(n+1)` is just `O(n)`.
- `O(n-1)` is also just `O(n)`.
- `O(n^2 + 3n + 8)` is just `O(n^2)`, because if `n` is big enough, `n^2` will become massive compared to just `3n`.
- `O(3n^3 + 4n^2)` is just `O(n^3)`.

For clarity, here's another way of looking at the second example, which is that `O(n^2 + 3n + 8)` is equivalent to `O(n^2)`. You can think of `n^2 + 3n` as `n` times `n + 3`.  The first `n` gets multiplied by both sides of the plus operator.  (3 times 2+5 is 21, the same as 3x2 + 3x5.)  Since we've already seen that `O(n+3)` is the same as `O(n)`, that means that `O(n(n+3))` is the same as `O(nn)`, better known as `O(n^2)`.


### But why?

Still with me?

The whole idea of Big-O asymptotic notation is that we can reduce a program's performance characteristics down to just one thing: how badly am I screwed if Facebook buys us out tomorrow and runs my program on a list of one billion users?

Going back to the example in the introduction, the `O(n^3)` program ran in around 58 days, and the `O(n)` program in five seconds.  That's just on 1,000 users - never mind a million times that.  At that point, you're looking at something like 160,000 years for the `O(n^3)` program to terminate, while the `O(n)` program can still finish its job in, funnily enough, 58 days.  I imagine I'll still have a Facebook profile in two months, but the human race may not even exist any longer in 160,000 years, leaving only a cloud of memes and memories to mark its passing.  See, asymptotic notation is important.

This is why we can so casually ignore the smaller terms and the constant multiples.  On a grand enough scale, they're irrelevant.  If my program is going to tell Facebook some useful information about its user base aeons after they've all died in the robot uprising, it doesn't really matter to me whether it takes 58 more days on top of that, or even whether it takes three hundred thousand years instead of a mere hundred and sixty thousand.  Likewise, spending 58 days to serve up a web request is just as bad as spending 232 days, in the real world.

This is why `O(n)` and its ilk are useful. Because if you don't respect them, you'll find your programs terminating after the heat death of the universe, and wondering what went wrong.


### OK, so how do we actually use it?

We can now read and write Big-O asymptotic notation, but why and when would we do that?

As you've probably gathered, this notation is used to evaluate algorithms for performance on arbitrary-sized data.  It's useful as a guideline, as a way of exploring why a piece of code might be slow, or for fleshing out one's understanding of code in general.

If you're concerned about the performance of a piece of code, or purely interested, finding a good Big-O estimation of its complexity is worthwhile.  Knowing the asymptotic execution time of library methods you call is also useful - a good rule of thumb is to be wary of anything with `O(n^2)` or higher execution time, and avoid `O(2^n)` or higher like the absolute plague.  If you thought our `O(n^3)` example from earlier took a while to iterate over 1000 items, an `O(2^n)` algorithm running the same 200 iterations per second would terminate a length of time roughly equal to the age of the universe _with another 281 zeros written after it_.

I wasn't kidding about the heat death thing.

(The universe is a bit of a wuss, and is only 13,798,000,000 years old, give or take a bit.  That's right, the age of the entire universe is far too short to usefully compare to our `O(2^n)` algorithm chugging along at 200 operations per second on a list of one thousand users.)

Some typical asymptotic run times for common methods in Python [live here](https://wiki.python.org/moin/TimeComplexity).  Other languages will probably perform similarly, but it's worth looking them up to be sure!

Notably, copying lists and the `in` operator are `O(n)` (because they have to look at all elements) and sorting a list is `O(n log n)`.  `log` is the opposite of an exponent, so sorting using the standard Python function is better than `O(n^2)` and worse than `O(n)`.  Binary search, which is an optimised way of checking if an element exists in a sorted list, runs in `O(log n)` time.

These are worth watching out for.  If you run a loop that checks whether an element is `in` another list, you're looking at `O(nm)` where `n` and `m` are the lengths of the two lists.  This is comparable to `O(n^2)`, and should be treated with similar caution if you think the lists might be large.  When performance becomes a concern, check over your code and make sure you aren't running any operations of `O(n^2)` or worse.  If you are, consider rewriting them so that they have a lower asymptotic complexity, and their performance will probably improve notably!


### A simplified example

Suppose we have some bad performance on a Python method that returns a list of yet-unused user ID codes.  We have a store of pregenerated 12-digit codes that can be sent out to users, and we can compile the list of used codes pretty easily by looking at the users themselves.  The method performs badly in tests, so it's not just the database queries required to fetch the data that are causing problems.

```python
def get_unused_codes(all_codes, users):
    user_codes = [user.code for user in users]
    return [code for code in all_codes if code not in user_codes]
```

This is typical-looking Python, using list comprehensions to do iteration.  To get a better angle on where the performance hit is, let's expand them out into for loops.

```python
def get_unused_codes(all_codes, users):
    user_codes = []
    for user in users:
        user_codes += [user.code]

    result = []
    for code in all_codes:
        if code not in user_codes:
            result += [code]
```

Let's call the number of codes `n` and the number of users `u`.  We could in theory have as many users as there are codes to give out, but not necessarily, so `n` is at least as big as `u`.

Appending to a list [is `O(1)`](https://wiki.python.org/moin/TimeComplexity), so the first for loop performs this "constant time" operation `u` times.  That makes it `O(u)`.

The second list performs `n` iterations over what looks like a simple bit of code, but it adds a conditional.  It so happens that this conditional uses the common `in` operator, which as we saw earlier, runs in "linear" time - it's `O(u)`.  This means that the second loop is `O(nu)`.  If `n` and `u` are quite large, this could be a very expensive loop indeed.  At worst case, it's `O(n^2)`, which we've already seen is something to be a little wary about.  In our original example of 1000 users and `O(n)` taking five seconds, `O(n^2)` takes an hour and 23 minutes.

We've established where the problem probably is.  How do we improve it?

One way would be to try to improve the loop's asymptotic complexity.

Fundamentally, our algorithm wants to look at every item in one list and check for its presence in the second.  Looking for every item is a bit hard to get round - that's going to be `O(n)` no matter what you do - but can we speed up checking for presence?

[Binary search](http://rosettacode.org/wiki/Binary_search) is well-known to be `O(log n)`. Rather than looking through the list linearly, you halve the size of the section you're searching in on each iteration. This leads even searches on very big lists to terminate pretty quickly.  The only caveat is that the list needs to be sorted.

However, if we did have a sorted list of user codes - which Python can do for us in `O(u log u)` time - then we could use a binary search on each iteration of the second loop instead of a linear search, turning `O(nu)` time into `O(n log u)`.  This means that even for an extremely popular website, generating the results of the method wouldn't take all that much longer than simply printing all of the codes, available or otherwise.

Python doesn't provide a native binary search function, but writing one or adapting a library function like [`bisect.bisect_left`](http://stackoverflow.com/questions/212358/binary-search-in-python) shouldn't be too difficult. With that in hand, we can make the change fairly easily.

```python
def get_unused_codes(all_codes, users):
    user_codes = [user.code for user in users].sort

    result = []
    for code in all_codes:
        if binary_search_in(code, user_codes):
            result += [code]
```

or, more compactly

```python
def get_unused_codes(all_codes, users):
    user_codes = [user.code for user in users].sorted()
    return [code in all_codes if not binary_search_in(code, user_codes)]
```

The time complexity of this method is `O(n log u)`.  Why?

The first line performs a list comprehension we've already seen is `O(u)`, and a sort, which we can [look up](https://wiki.python.org/moin/TimeComplexity) and discover is `O(u log u)`.  The latter is larger than the former, so we can ignore the extra `u` operations thanks to the way big-O notation is constructed.

The second line performs an `O(log u)` operation `n` times, for an asymptotic runtime of `O(n log u)`.  Because `n` is at least as big as `u`, this dominates the `O(u log u)` term from earlier, and so the overall time complexity of the method is `O(n log u)`.

It doesn't look like much, but that is a big improvement.  For our 1000-item long data set, an `O(n log n)` algorithm would take around 50 seconds to complete, whereas the `O(n^2)` algorithm was 93 minutes.  Because `log` grows so slowly, we're still pretty safe even if the number of users increases tenfold.


### TL;DR

To say that a piece of code is `O(E)` means that:

- As long as `n` is at least a certain, constant size,
- the code takes no more than `kE(n)` iterations to run.

This is very important because physics.
