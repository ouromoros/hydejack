---
layout: post
title: The N-queens Problem in Haskell
tags: [haskell, programming]
---
It suddenly comes to me that writing a Haskell solution for the N-queens problem would be very good practice. Since an efficient solution isn't very obvious (for me), I decide to do some explorations.

## Naive...

The most naive solution would be to just iterate through all the possible iterations and check one by one.

~~~haskell
queen :: Int -> [[Int]]
queen x = filter isSolution $ iterations x

iterations :: Int -> [[Int]]
iterations x = iterations' x
  where iterations' y | y == 0    = [[]]
                      | otherwise = [a:tail |
                                     tail <- (iterations' $ y-1),
                                     a <- [1..x]]

isSolution :: [Int] -> Bool
isSolution s = isSolution' s 0
  where isSolution' s c = case s of
          [] -> True
          x:xs -> not $ or [x `elem` xs,
                            any (\(a,b) -> a + b == x + c) (zip xs [c + 1..]),
                            any (\(a,b) -> a - b == x - c) (zip xs [c + 1..]),
                            not $ isSolution' xs (c + 1)]
~~~

Pretty neat, right? Unfortunately computing the result of `queen 8` consumes **55,192,257,624 bytes** of space. Like, seriously? I don't remember having that much memory. But if ghci says so, then so be it.

Also, it takes **46.49 secs**, which is an alias of *forever* for such a simple task. So there is much space to improve, and let's see how.

## Improved...

So the problem is while it is very pleasant to see all the things divided and modular and adhere to some good design patterns, it's just not practical. We go through all the possible iterations (n to the power of n! ), which is a really horrible number, and we start over when we calculate many iterations that actually have similar conflicts (in the first several columns).

If we want to be efficient, we must filter the solutions while generating solutions. In other words the checking and generating function must be combined. Below is the improved version.

~~~haskell
queen :: Int -> [[Int]]
queen x = queen' x
  where queen' y | y == 0    = [[]]
                 | otherwise = filter isSolution
                                      [a:ps | ps  <- queen' $ y-1,
                                                a <- [1..x]]
        isSolution s = case s of
          [] -> True
          x:xs -> not $ or [x `elem` xs,
                            any (\(a,b) -> a + b == x) (zip xs [1..]),
                            any (\(a,b) -> a - b == x) (zip xs [1..])]
~~~

We see that we do a filter each time we generate *part* of the solution. Notice we exploit the fact that the first few columns are actually "independent" from the others, so we can do filter this way. I'll show you the running time:

| N    | Memory/bytes   | Time/secs |
| ---- | -------------- | --------- |
| 8    | 107,112        | 0.00      |
| 10   | 665,681,320    | 0.54      |
| 12   | 20,111,023,696 | 17.11     |
| 14   | ...            | ...       |

 We can see that it's blazing fast now! It's 0.00 sec when N equals 8, which means we have over **10000x** boost in this edition. It's incredible that the code is actually shorter than before, that's why I like programming.

## Finale

Unfortunately I'm not patient enough for the program to finish computing the case when N equals 14. Hopefully I can come with more efficient solutions later on.
