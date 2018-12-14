---
layout: post
title: 'Implementing Lazy List in Python'
tags: [Programming, Python]
categories: [English]
---
# What We will Build

In this article, we will implement a lazy list that will be much like `Data.List` in Haskell-- actually it'll share most of the API with the Haskell list. Don't you worry about it. We're using Python here, and you're not expected to have any experience with Haskell.

What is a lazy list? Before we enter the main topic here, I'll first clear it up for some of you, so you will at least have *some* clues to what we're going to do. (safely skip to the next topic if you already know what a lazy list is)

First, I'll explain what is laziness. Imagine we have the following code:

```python
def f(a):
    b = list(g(x) for x in a)
    return b
def main():
    a = [1, 3, 5, 8]
    b = f(a)
    c = f(b)
    print(b[2])
```

where`a` is a list, and `g(x)` is some arbitrary function that does whatever it wants to an element in `a` and returns a value. Presented with this piece of code, a programmer equipped with sharp eyes will observe the following:

1. We never used `c` in the code, but still we computed it for every element in `b`.
2. We only make use of `b[2]` even though we've computed `g(x)` for every element in `a`. If `g(x)` is expensive, the overhead here is unacceptable.

The presented code can be easily optimized by removing the unnecessary computations, even smart compilers might be able to do the optimization for you. But in reality things can grow trickier: Sometimes we will want to access multiple elements in `b`, sometimes we will want elements in `c` or more complex structures, and we don't know which element we want until runtime... Of course, there's always the option of optimizing them by hand, but can they somehow be magically resolved so we programmers don't have to worry anymore? The answer is yes: with the introduction of **laziness**.

Laziness is a description of the program's behavior: we don't evaluate the value of a statement until it is actually needed. For example, the execution of the above code will be like the following:

1. `a` is defined.
2. `b` is defined, OK. I'll compute it if it is needed afterwards.
3. `c` is defined, OK. I'll compute it if it is needed afterwards.
4. Oops, you are requesting the value of `b[2]`. Let me see.. according to the previous instructions it should be the value of `g(a[2])`...Yes, I've got `g(x)` and `a[2]`....OK, it is here. Just print it.

I've somehow over-humanized the process, but this is how it is like. The program doesn't do what it is told to do immediately— it only does so when the value is actually *needed*. And then, with a lazy list, we would be able to do all sorts of things... I'll show them to you in the end.

The complete code for the project can be find [here](https://github.com/ouromoros/lazy-list-python) on Github.

# First Step

First let's review what a lazy list is like: rather than just store computed results, it only compute needed elements *lazily*. In order to achieve that, it should be able to keep the information how each element of its should be like, and *evaluate* them when needed. To implement one, the key problem would be to determine how we keep the information of *how the elements should be computed*.

What we desire here is a way to get the value of an element given its index `i`. Naturally this leads us to functions: a function with an argument `i` and returns the value at that index. We will also need a way to *cache* the result of this function since we don't want to evaluate the value every time. Here we just use a `dict` for that purpose. Finally our draft version of `LazyList` would be like this:

```python
class LazyList(object):

    def __init__(self, func):
        self.func = func
        self.cache = dict()
    def __getitem__(self, key):
        if key in self.cache:
            return self.cache[key]
        self._evaluate(key)
        return self.cache[key]
            
    def _evaluate(self, i):
        self.cache[i] = self.func(i)
```

where `func` is the function that can get the value elements for us, and `cache` is a dictionary that keeps the computed values so we won't need to compute them again if we need the same value in the future.

Now with `LazyList` defined, we can already define a few handful lists and functions:

```python
# list of natural numbers
naturals = LazyList(lambda i: i)
# list of zeros
zeros = LazyList(lambda _: 0)
# list of ones
ones = LazyList(lambda _: 1)


def make_lazy_list(iterable):
    """returns a LazyList made from iterable"""
    copy = list(iterable)
    return LazyList(lambda i: copy[i])


def repeat(item):
    """returns an infinite list whose elements all equal to item"""
    return LazyList(lambda _: item)
```

Note that `naturals`, `zeros` and `ones` are all lists with infinite elements! Now we already have something that ordinary Python list doesn't support. However, these functions and lists alone aren't really of any use. The true power of `LazyList` will reveal itself only with the presence of a bunch of higher-order functions. 

# A Taste of Function

You've probably already noticed the use of `lambda`s in the code above. They're just shortcuts for defining anonymous functions. Fortunate for us, functions are first-class citizens in Python, which means that we can pass them around like any other object. This brings a lot of flexibility to us.

First let's implement the probably most used function— `map`. Yes, this `map` is almost the same as Python's built-in `map`, except that it returns a new `LazyList` rather than modifying the called `LazyList`. Our `LazyList` is immutable viewed from the outer world. Actually, a list which is both lazy and mutable makes no sense. I'll leave that for you to think.

```python
class LazyList(object):
    
    ...
	
    def map(self, f):
        return LazyList(lambda i: f(self[i]))
```

The implementation is very straightforward and doesn't really require any thought at all. With `map` at hand, we can do various things. Our code at the start of this post can be changed to this:

```python
def main():
    a = make_lazy_list([1, 3, 5, 8])
    b = a.map(g)
    c = b.map(g)
    print(b[2])
```

Now the list is lazy and we don't have to worry about doing more-than-necessary computations! `g(x)` will only be called once on `a[2]` and it's done automatically. Great!

We can also implement `take`, `drop` in this way, but that's about where it ends. To implement functions like `filter`, `append`, `concat` or `tails`, our naive implementation here is just not enough. We will need to keep track of size information and treat infinite lists as a special case. We'll explore them in detail in the next post.
