---
layout: post
title:  "Non-physicality in Physics Simulations"
date:   2024-10-10 14:44:33 +0200
categories: simulation physics thermodynamics mechanics
---
# Under Pressure

## Finding Time

The first thing I wanted to achieve before really diving into taking measurements was to establish a way to measure over a
consistent and repeatable timescale, ideally without having to introduce some sort of explicit timer into the simulation.
Just measuring the state of the system after each collision would be possible, but not only would this occur extremely frequently
for many simulations, the asociated time scale could vary a lot: although most collisions happen almost instantaneously, some
take a really long time as the simulation sits in a period of relative 'calm'.

Luckily, 2nd-year statistics comes to the rescue in the form of the central limit theorem: if we sum up a number of identical
random variables, provided they are independent of each other (and probably, some other necessary condition) the random variable
that gives their sum will converge to a normal distribution. As a result, our first problem is solved by taking measurements
every (for example) 50 collisions:

![the central limit theorem in action]({{ site.url }}{{ site.baseurl }}/assets/images/hundred_collisions_dist.png)
