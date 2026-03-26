+++
date = '2025-08-15T20:18:10-06:00'
title = "World's Worst Compression"
description = "Slowest compression there ever was."
tags = ['machine-learning', 'math', 'search']
draft = false
+++

# Context
I actually came up with this awhile ago for a school project (search and
planning class). For an replacement grade we had to just apply for some of the
methods we had learned about in class. At the time for reasons I will not go
into here I was thinking about efficient ways to store mesh data on GPUs. That
means considering how to minimize mesh memory to ensure quick transfer from RAM
to VRAM, appropriatly reducing mesh detail according to distance from camera and
resolution, etc etc. I was simultaneously annoyed by the fact that mesh data and
collision data was not unified. Then I had an idea. An awful idea. I had a
wonderfully awful idea ⊂(◉‿◉)つ. "What if all my data was defined by functions?
What if all my data was defined by SDFs? Or most of my data was defined by
functions? Then I get arbitrary resolution on those object, convenient normals
as long as I restrict the operations I permit, and I can kabe my kace and tea it
too."

So I decided I would make a project out of a simpler version of this:
compression for terrain data. Training a neural network was insufficient for my
task because &mdash;well there wasn't really a reason. I had to use search
algorithms and pin holed myself into syntesizing a program&mdash; no there was a
reason: I wanted to be able to take derivatives of the final function. Still
neural networks are probably nicer because matrix multiplication is really fast,
and choosing a good activation could yield some interesting results.

# Overview

Consider a height field (terrain map with no overhangs/caves; z is function of
x and y) mesh of uniform density. The worst way to store is with a point in
RR^3 for each vertex and then a list of edges/faces. To reduce the storage
requirement you can do some obvious things like inferring the (x,y) from the
position of the height in the buffer (assuming that points are distributed
uniformly in a grid). Or you could remove the uniformity constraint and use
more vertices where the derivative of the surface changes (more detail on not
flat regions).

But we're going to do something strange instead and find a function that fits a
terrain mesh to give it "infinite resolution." Its not in practice because
there is a finite number of representable floating point numbers. 

# Representation

The first time I did this I decided to make a stack based interpreter for an
array of constants, variables, and operations layed out in Reverse Polish
notation. Relative to an AST it was much faster to evaluate being fully
contiguous and quick to iterate over. It wasn't very nice to search over
because it was hard to check if the arity of the various operations was being
respected.

I ought to I read a paper a couple months ago "[DreamCoder: Growing generalizable,
interpretable knowledge with wake-sleep Bayesian program
learning](https://arxiv.org/abs/2006.08381)" and they used lambda calculus
expressions to represent programs in a search space.

Wait should I do AST?
