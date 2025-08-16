+++
date = '2025-08-15T20:18:10-06:00'
title = "World's Worst Compression"
description = "Slowest compression there ever was."
tags = ['machine-learning', 'math', 'search']
draft = false
+++

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
