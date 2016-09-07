---
layout: post
title: "Associative In Haskell"
date: 2015-11-30
categories: haskell,functional programming
---

Well, associative is a basic concept in math. Simplify, (1 * 2) * 3 = 1 * (2 * 3).

So when I was working on the "write yourself a scheme in 48 hours", there was an exercise which blocked me. The exercise looks easy. Just get inputs from console and compute the sum of the two inputs, then prints result on the screen. And there is a line in the answer -- "print ((read $ args!!0) + (read $args!!1))".

The "$" really bothers me. As the explaination of wiki, "$	0	right	infix function application (f $ x is the same as f x, but right-associative instead of left)".

So what does this mean?

It just means you can drop the "$", instead of "print ((read (args!!0)) + (read  (args!!1)))".

The "associative" concept comes with precedence togather. I think for computer execution, expecially for haskell compile or run, it cannot work it out when you get a lot of parameters or instructions. It's not like "1 + 2 * 3", because the precedences of "+" and "*" are different. So if you remove the "$", like "print ((read args!!0) + (read args!!1))" and compile, you will get an error. Just because the ghc doesn't know that it should do "args!!0" first. Actually, ghc thinks that it should take "read args" first, and then "!!0" which cause the error.

When you use a "$", it turns to be right-associative. Then the ghc only matches the "args!!0", not "0" or "!!0".

Well, functional programming looks fun. Pretty cool, but totally different from OOP. Still figuring out how a project developed by haskell works out.
