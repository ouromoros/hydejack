---
layout: post
title: Writing a simple evaluator for Forth in Haskell, Part I
tags: [haskell, programming]
---

In this post I will introduce the basics of *the Forth programming language* and provide the basic skeleton of a simple Haskell evaluator we're going to write.

## Table of Contents
{:.no_toc}
0. table list
{:toc}

Jump straight to the next section if you don't want to waste time reading my completely irrelevant personal records.
{:.message}

So I'm learning Haskell . It all began about a month ago. It's not a big deal, *except* that apart from the idea of functional programming I'm already familiar with, I still get overwhelmed  by a large number of new concepts (e.g. type inference, laziness, monads, *etc*).

But I will not call it the end. After struggling through the fantastic book *Learn You a Haskell for Greater Good* and reading and thinking a lot more, I feel obliged to get some practice (The only way to *learn* something), and I did.

As a ~~lazy~~ efficient person, I usually do exercises on platforms where the test environment is already set up neatly. This time I chose to do exercises on exercism.io, a great website where you can fetch the exercise, implement it, pass all the tests, and submit it when it fits you.

So after passing the first couple of exercises, with my growing confidence, feeling bored with the triviality of passed ones, I jumped right to the bottom of the list of exercises and clicked into this Forth thing [here](http://exercism.io/exercises/haskell/forth/readme). Now *that's* where it all starts, where I discover my own immaturity.

## What is Forth?

> **Forth, the computer language, was created for programming embedded and real-time applications.** Today, it is available for developing applications on Windows, DOS, and variants of Unix that include macOS. Additionally, commercial-grade Forth cross compilers generate highly optimized code that runs on a variety of microprocessors and micro controllers, and prove themselves very capable in custom-hardware environments.

The above is the official definition of the Forth programming language. Ok, if you're like me, you would have completely no idea about Forth before and after reading that, ~~actually I just decided put it here to waste your time~~.

While Forth seems to be a very customizable and powerful language (honestly, I have no idea), I'm only going to implement very limited features here. So basically you only need to know several things about Forth:

* In Forth you work with a **stack**. Basically you just manipulate the stack with different commands. You can also define your own command. The commands don't have arguments. You can say that they take the things on the stack as arguments and then pushes the result back on the stack.
* To keep things simple, we are only going to support signed integers on the stack here.
* We evaluate the program one line at a time. Each line would contain multiple commands or could be a definition of a new command (could overload existing one).
* The basic commands we are going to support are the following:
* - Number.: `1`, `289`, `1024`, etc. A number gets pushed on the stack and does nothing else.
  - Arithmetic commands: `+`, `-`, `*`, `/`. These are pretty self-explanatory. They take the first two things on the stack and compute the result which is put back to the top of stack. One thing to notice here is that the **top** item is treated as **second** argument here.
  - Stack manipulation command: `DUP`, `DROP`, `SWAP`, `OVER`. `DUP` just duplicates the top item on the stack. `DROP` drops the top item. `SWAP` swaps the top two items. `OVER` duplicates the second item on the stack and pushes it to the top.
  - Defining a new command: `: word-name definition ;`. Word-name is the name of the new command, followed by the definition of the command which is typically a combination of multiple other commands.

So finally we're ready to go now. If you don't get it, that's okay because you can always look back when things get confusing.

## First attempt

So at first this all seem a very simple task (and it is). You just keep a stack and mapping of commands and do everything. Without much thought, I just started writing the program as usual, hoping the implementation details would reveal themselves as I write on.

Yet they didn't, and I got stuck. It seemed that all things I could think of are procedural calls and mutable data structures, and I could hardly write a few lines. It lasted quite a while before it dinner time, then only after I managed to sort things out during the time not in front of my computer did I begin to actually work on it.

So that's when I realized the difference of programming in functional language. You have to have a view of the whole program before you can actually work on it. When writing in other languages I just write on the fly and cut-and-paste code when it fits me, which I have to say ends up wasting a lot of time. Haskell forces you to *know* what you are writing because it is just so difficult to write, which actually turns out to be a good thing.

## Outline

In the evaluator, we're going to take the input, parse the operations, pass the  initial state through the operations and then return the final state. Pretty simple, right? Here we go.

So first we're going to define the *state* of our evaluator. Basically to evaluate a line of code, we only need to keep track of the stack and previously defined operations. It is pretty straight forward:

~~~haskell
data ForthState = ForthState{stack :: [Int]
                             ,ops :: (M.Map String Op)}
~~~



As is indicated above, we're going to use `Map` from `Data.Map` to store a mapping from names of operations and its underlying representation.

Here is the definition of `Op`:

~~~haskell
type Op = [BaseOp]
data BaseOp = Int Int | DUP | DROP | SWAP |
              OVER | MUL | MINUS | ADD | DIV | DEF String Op
~~~

We define `Op` as a list of `BaseOp` here. The advantage of this is that we can easily combine different `Op`s to form new `Op`, which is just what we do when defining new operations.

In the expression `Int Int`, the first `Int` is a new constructor that we have defined for `BaseOp`, and the second is just the normal `Int` type.
{:.message}

The definition of `performOp` and `performBaseOp` follows naturally:

~~~haskell
performOp :: ForthState -> Op -> Either ForthError ForthState
performBaseOp :: ForthState -> BaseOp -> Either ForthError ForthState
~~~

Here `performOp` takes a `ForthState` and then call `performBaseOp` to traverse the `BaseOp` list and apply it to do a series of `transformation` of `ForthState`. Since exceptions can happen at any time we perform an operation, an `Either` structure is used for the returning type.

`ForthError` is defined as followed, the names are pretty self-explanatory.

~~~haskell
data ForthError
     = DivisionByZero
     | StackUnderflow
     | InvalidWord
     | UnknownWord Text
     | OtherError
     deriving (Show, Eq)
~~~

Now I'll throw you definitions of the remaining functions we're going to have. Don't worry, I'll explain them one by one.

~~~haskell
parseText :: String -> ForthState -> Either ForthError Op
empty :: ForthState
evalText :: String -> ForthState -> Either ForthError ForthState
~~~

`parseText` takes a line of operations (can contain both `BaseOp` and `Op`) and parse them into a list of `BaseOp` (an anonymous `Op`). `ForthState` is needed here since the mapping of user-defined operations is needed here.

`evalText` takes a `String` and the initial state, evaluates it and return the result. It really acts just as a glue between `parseText` and `performOp`.

As expected, the return type of either function is a `Either`, since exceptions can happen.

## Brief Summary

Let's see what we have achieved so far.

1. We have got an idea of what *the Forth programming language* is.
2. We have written a skeleton of the evaluator we're going to write.

Here is the code we have (only skeleton):

~~~haskell
data ForthError
     = DivisionByZero
     | StackUnderflow
     | InvalidWord
     | UnknownWord Text
     | OtherError
     deriving (Show, Eq)

data ForthState = ForthState{stack :: [Int]
                             ,ops :: (M.Map String Op)}
                             
type Op = [BaseOp]

data BaseOp = Int Int | DUP | DROP | SWAP |
              OVER | MUL | MINUS | ADD | DIV | DEF String Op
              
performOp :: ForthState -> Op -> Either ForthError ForthState

performBaseOp :: ForthState -> BaseOp -> Either ForthError ForthState

parseText :: Text -> ForthState -> Either ForthError Op
empty :: ForthState

evalText :: Text -> ForthState -> Either ForthError ForthState
~~~

Next time we're going to implement the details of these functions (which is really not so hard). Just be prepared.

[~~Writing a simple evaluator for Forth in Haskell, Part I~~]()
{:.faded}
