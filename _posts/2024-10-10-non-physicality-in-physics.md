---
layout: post
title:  "Non-physicality in Physics Simulations"
date:   2024-10-10 14:44:33 +0200
categories: simulation physics thermodynamics mechanics
---
# Non-physicality in Physics Simulations

*This blog post is part of a larger series following the development and results from the `eight-ball` simulation.
If things here seem confusing or out of context, consider having a look at the rest of the series (which, admittedly,
might not exist, yet).*

## Introduction: spotting non-physicality

The first real bit of data I wanted t extract from the simulation was the time that elapsed between each collision.
The reasons for that were two-fold:
- first, this was an easy starting point, because getting the system's global time at each collision was relatively
straight forward.
- second, I was hoping that the time between consecutive collisions might behave like some sort of sensible Poisson
distribution, i.e. most collisions happening at a well-defined time; a rapidly shrinking number occuring with less
elapsed time; and a slowly shrinking number happening over longer time scales. Importantly, I was hoping that I
could then sum over say ~100 collisions and hence get a well-defined time interval, because by the Central Limit
Theorem (CLT) the summed Poissonian variables should collapse to a Gaussian with a pretty small variance (in fact,
them being Poissonian is pretty irrelevant here, but I should probably check the details of what makes a variable
IID or so before I get basic statistics wrong on the internet).

Anyway, the initial objective was simple enough: collect a large number (say, a million) of collisions and just look
at the time elapsed between each. Bin the elapsed times into some sort of histogram and then check that the distribution
has the expected shape. To jump the gun to when this later worked out, this is what we were going for:

![good distribution, not what we expected](./../_images/successful_times_dist.png)
