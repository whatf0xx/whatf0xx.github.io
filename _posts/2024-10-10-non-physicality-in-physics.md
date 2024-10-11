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

![good distribution, not what we expected]({{ site.url }}{{ site.baseurl }}/assets/images/successful_times_dist.png)

*Ok, that's not a Poisson distribution, but this is science so we roll with the punches.*

Although the behaviour for low times isn't what we expected, the behaviour in the tail is heuristically correct, as in it's very
rare indeed for a lot of time to pass with no collisions taking place. What is interesting for now is the behaviour I observed
when I first ran this simulation to count the collision times:

![what's going on here, then?]({{ site.url }}{{ site.baseurl }}/assets/images/four_balls_times_dist.png) 

Apparently, a **huge** number of collisions are occuring with exactly the same time in between. Given that the dynamics of
the simulation are pretty stochastic and constantly changing, this is really weird. What's even more strange is that this
oddity is superimposed on top of what otherwise looks like quite a sensible distribution of times (we even recover *some* sort
of Poissonian shape). So, what gives? Why is our simulation showing what seems to be non-physical behaviour?

## Detective Work

Playing around with the simulation revealed that this behaviour was only happening for larger numbers of collisions (~3 million,
as opposed to just 1 million), which suggested that at some point the simulation was breaking down. Using the `comic_strip()`
method I wrote as an early heuristic to check the simulation was doing something sensible for the first few collisions, I was
surprised to see apparently empty plots in the place of my mock simulation chamber. Upon further inspection, I realised that
these weren't empty, rather, they were super zoomed out, with balls a very long way away from the container they were supposed
to be confined to!

It seemed as though, at some point, the balls had one-by-one clipped through the container wall until only one remained inside.
With just a single ball bouncing around, the time between each collision became very stable, as one might expect from the
simpler dynamics. The question was, how were balls clipping through the container in (what I thought was) my robust simulation?
Why did I bother to write all that Rust code if it was going to work so poorly?

## Pinpointing the Moment of Judgement

To work out the moment that things were going wrong, it was time to introduce a new measurement to our simulation: the positions
(and, importantly, their distance from the origin) for each ball. With this, we could check over all the balls, and notice
the moment when one travelled more than a distance *R* (the container radius) from the origin.

I wrote a quick and dirty version of this which would just cause a thread panic when this occured, and let us know how many
collisions had occured up to this point. Running this with the parameters from before, it was possible to see that a ball ended
up more than *R* m away from the origin on the 2 115 611-th collision - not a bad number, to be fair, but not really as
bullet-proof as I would have liked. 

Around this point, I also noticed that I had been running the simulation with only 4 pretty large (about 1/20th the container
radius) balls. Although I am still unsure, I think this might point to a potential reason for the breakdown in the simulation's
accuracy for large number of collisions. I'll explain in a second.

First of all, though, having pinpointed when the simulation was breaking down, it was possible to use the `comic_strip()` method
again to have a look at the dynamics around that point:

![There it is!]({{ site.url }}{{ site.baseurl }}/assets/images/moment_of_truth.png) 

*(Excuse the lack of axis headings, this method was for debugging)*

## What are the Chances!?

The graphic above goes some good way in explaining how the clipping occurs: focus on the two balls off to the right, next to the
container edge. In the first panel, the two balls collide, just *before* the right-most one reaches the container edge (the arrows
for the balls' velocities show the new velocities *after* the collision). Then, the right-most ball reaches the container edge just
a moment later and heads back for the container's centre. However, note the timestamps for the second and third collision: to
within the 64-bit floating point accuracy of the simulation, they happen at the same time! As a result, when the next collision
occurs immediately after, the simulation doesn't then consider that the right-most ball and the container should then collide; an
early design choice was that collisions 'in the future' but happening after 0 time shouldn't be counted as collisions, because this
requires the balls to be relatively stationary - unless, of course, they are part of a three way collision like this.

## So, what to do?

One option at this point would be to go back and allow the system to add in new collisions that are happening 0 seconds in the future,
but before we consider that too deeply I think it bears doing some quick analysis on why this occured in the first place, and, quite
importantly, why this doesn't seem to happen with a large number of balls even for a really high number of collisions.

Note that in the 4-ball case, the order of magnitude of time between collisions is about a second. That means that for about 2 million
collisions to occur (where we notice clipping behaviour) about 2 million seconds elapses, which is pretty spot on for the timestamps
on the comic strip. On the other hand, with a very large number of very small balls whizzing around, most of the collisions are
occuring at well under a millisecond. Hence, to generate a similar number of collisions requires on the order of 1/5000th or so the
time.

What (I think) is important about this is that in our floating point mantissa to store the global time, about 12 extra bits are being
saturated storing (mostly) useless information about the amount of time that has elapsed. Turning this on its head, if the global
time has reached about 2 million, then the first 21 bits of the number are just storing the part that is greater than unity. Then,
given that to within 5 d.p. our collision happens at the same time, we need another 15 bits to store that. In fact, if we actually
print the whole values for the times at which the collision occurs, we see they are the same to 10 d.p., or over 30 bits in the
floating point representation! This explains why the simulation couldn't distinguish them: in reality, time always elapses between
collisions, but with only 52 bits to store the relevant information, 64-bit floats just aren't precise enough to pull this off.

The question is, if the system clock stayed lower, would this make enough of a difference to avoid these clipping errors? For the
case that worked so far, we had 12 extra bits, and hence no floating-point saturation. One way to test this, then, would be to 
let the simulation that is currently behaving run for 5 000 times as long and see if some clipping occurs towards the end, as the
clock saturates. However, it might be the case that with the significantly smaller balls, the chances of a triple collision become
much smaller; I'm not sure, it sounds like some sort of fairly involved probability question that is way beyond the scope of this blog.

## Conclusion

So, later, I might extend the running time to see if we get this sort of saturation again with the more realistic (as in, small
balls compared to the container size, like gas particles) parameters. For now, I think the way to go is to keep working with these
heuristics - such as checking that the balls remain within the container; and keeping an eye on graphs for when physical simulations
start looking very non-physical. 

