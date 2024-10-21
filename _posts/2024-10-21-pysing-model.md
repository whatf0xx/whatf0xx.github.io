---
layout: post
title:  "The Pysing Model"
date:   2024-10-21 09:35:00 +0200
categories: simulation physics thermodynamics ising
---
# The *Py*sing Model

## Motivation

Back in undergrad Physics I really enjoyed the courses in statistical mechanics, and I think a large part of that was due to the
cool patterns and shapes that matter (or systems) around a critical temperature produce. Much like watching a fallen drop of dye
expand through a sheet of thin paper, I was fascinated by the fractal, winding shapes that a 2D-Ising model could produce at the
critical temperature.

Although the maths of mean-field theories and renormalisation groups was cool (read: mind-blowing) I always wished I had some way
of visualising these systems, rather than just calculating their (at times sterile-seeming, by comparison) physical properties.

Looking back, a fairly obvious solution to this would have been to have created a percolation solver/visualiser: at a critical
percolating probability, these systems also produce the same sort of shapes that I wanted to see in an Ising model. Equally, I
think this would have been slightly less physically satisfying: one thing I really wanted was to watch criticality emerge as a
system quenched from an initial random state.

## Breaking away from the Free Energy

Like a lot of these sorts of projects, before jumping onto Wikipedia to find a solution that I was sure must exist, I wanted to
see if I could come up with a way to generate these sorts of shapes myself, first. I cast my mind back to the final year of my
undergrad and returned with two pieces of information that would prove key:

### 1. The Ising model master equation

$$ U = -J\sum_<ij>{s_i s_j} - H\sum_i{s_i} $$

I am never one much for mathematical sentimentality, but I *do* like when a Hamiltonian tells you everything you need to know
about a system in such a clear way. Here, we are describing the system energy, *U*, in terms of all of the individual spins,
the $s_i$s.
