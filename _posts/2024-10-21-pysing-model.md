---
layout: post
title:  "The Pysing Model"
date:   2024-10-21 09:35:00 +0200
categories: simulation physics thermodynamics ising
---
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

$$ U = -J\sum_{<ij>}{s_i s_j} - H\sum_i{s_i} $$

I am never one much for mathematical sentimentality, but I *do* like when a Hamiltonian tells you everything you need to know
about a system in such a clear way. Here, we are describing the system energy, *U*, in terms of all of the individual spins,
the $$s_i$$s. There is immediately something quite satisfying about this to me, because we are going from a totally microscopic
approach (examining all the individual spins/domains in a lump of magnet) and arriving at a value for the energy of the whole
lump. Pretty cool.

Each spin can have a value of plus or minus one, depending on if it's facing up or down. Then, what the Hamiltonian tells us is this:

#### The right-hand term

*(You didn't skip, we started on this side for a reason)*

Remember watching metal flakes line up with the field lines on a bar magnet back in high school? This is what we are looking at
here. For each spin in the magnet, we check if it's aligned with an external field (*H*) or anti-aligned. Aligned is a
low-energy configuration, corresponding to $$s_i = 1$$, and the system energy decreasing, whereas the anti-aligned case will
correspond to $$s_i = -1$$, and the system energy increasing, each time by a step of *H*. Overall, this term tells us how well
aligned the domains of the magnet are with an external magnetic field.

#### The left-hand term

The left-hand term is more interesting, as it describes the spin-spin interactions within the magnet. In reality, each spin
'feels' the influence of every other spin - they all want to line up, in the absence of anything else going on (check: if 
$$s_i = s_j$$, how does this affect the system energy?). However, considering the interactions of every spin with every other
spin would lead to a vastly complicated calculation. As close-by interactions dominate compared to further away ones, we simplify
by only considering *nearest-neighbour* interactions: this is what is denoted by the *<ij>* under the sum. In terms of making a
simulated model, this simplifies things, too, because we have a well-defined way of working out these interaction terms: for each
spin, we just have to know which spins are closest to it in the system in order to calculate the interaction energy term. Finally,
*J* gives a scale to the interactions: we assume that they are the same in every direction (isotropic) and the same everywhere in
the magnet (homogeneous) so we can just use a single coefficient to characterist the internal interactions: *J*.

### 2. Minimising the system's Free Energy

Unlike systems that we study in 'cold' mechanics, in thermodynamics and statistical physics we are not interested in simply
minimising a system's energy. For any given configuration of a system's macroscopic properties (its energy, pressure, temperature,
magnetism, or even shape) there might exist a huge number of configurations of the system's microstates (what the atoms within
it are actually doing). Temperature is not such an easy thing to define, but one way of looking at it is in terms of how well a
system obeys simple laws about energy conservation: at 0 temperature, a system might not sit solely in a ground state, but the
state it sits in should be perfectly predictable from the system's energy. By contrast, at higher temperature, 'thermal
fluctuations' allow a system to occasionally work against energy conservation, sitting in a more evenly distributed set of energy
states than satifying the Hamiltonian might have allowed. So it is with our Ising model: although just minimisng the Hamiltonian
would lead to a magnet where all the spins were aligned with each other (and an external magnetic field, if it is present) always,
this is in reality not what we observe: magnets don't always behave as perfect, infinitely strong ferromagnets, and importantly
at *high temperatures* their magnetism degrades. Taken in view of statistical physics, we can explain this: although the minimum
eneryg state always wants perfect domain alignment, thermal fluctuations leads to pockets of domain misalignment which reduce the
system's magnetisation.

The way physicists treat statistical systems is hence by minimising the Free Energy, usually given by *H*, but here given by *V*,
so that it isn't confused with the external field *H*, above:

$$ V = U - TS $$

Now, if we try and minimise this quantity, we have to take into consideration the temperature *T* and the entropy *S*, described
below. Note first that we haven't totally changed what we are doing: at low temperatures, the second term in the free energy is
going to be less important and we basically get back to minimising the energy. As such, at low temperatures, systems go back to
behaving like they do in models that don't consider temperature, and just minimise the system energy.

On the other hand, at high temperatures, we have to consider this new term, which considers the system entropy. Entropy is a
quantity of macroscopic systems at a given **macroscopic** configuration (i.e. we know the system's energy, pressure,
magnetisation) which describes the uncertainty in the microscopic configuration this might correspond to. In general, this is
linked to the number of microscopic configurations that could correspond to the given macrostate. As such, for macrostates that
can be defined by only a small number of microstates (such as a very high magnetisation ferromagnet, where all the spins must be
aligned, leaving only the possibilities that they all align with an external field or all anti-align with it) the entropy is low,
whereas for macrostates which could correspond to a large number of microstates (low magnetisation where all we know is that all
the many spins sum to 0, in potentially a huge number of ways) the entropy is high.

This then gives us an indication of how our magnetic system is going to behave: when the temperature is high and hence we have
to take the right-hand term into account, we have to *maximise* the entropy in order to reduce the free energy over all. Hence,
at high temperatures, we favour macrostates with lots of microstates, such as low magnetisation.

### Back to Simulating pretty patterns

Phew, thanks for your patience. Initially, then, I thought that I would be able to come up with states that did a good job of
minimising the system's free energy. The trouble is, what I really wanted to do was to make pretty pictures of Ising model 
*microstates*, for which the free energy is totally undefined; to reiterate, the free energy is a property of a macroscopic
system, which doesn't care about the microscopic configuration, it only cares about the *number* of microscopic configurations
corresponding to the macroscopic properties.

Just as this lead seemed to dry up, the other (slightly less loved, I reckon) child of statistical physics appeared to the rescue:

### the Boltzmann Distribution

The Boltzmann distribution is the probability of finding a system in a given *microstate*, given some assumptions about the
isolation of the system and its macroscopic properties. Because it actually concerned system microstates, it immediately promised
to be a better lead for my simulation than the system free energy. Additionally, working with probabilities is nice: I don't know
exactly how my states should look, but if I can make them 'maximum probability' states according to a physical model I trust, I
will be quite happy with what they represent!

Here is the Boltzmann distribution for a given system microstate in the absence of information about the system's other properties:

$$ p(\{s_i\}) = \frac{e^{-\beta U(\{s_i\})}}{Z} $$

Here, *Z* is the partition function, which is generaly incredibly important for physical calculations but doesn't add anything to
this simulation, so we will henceforth basically ignore it by multiplying through the whole equation by a factor of *Z* and treating
it as a constant after that (which is true, at least, it won't vary with the changes to the microstates that we will be making).
$$ \beta $$ is the inverse temperature, which physicists use to avoid writing `frac{}{}` too much in LaTeX documents.

### The Plan

Generating an initial micrstate is easy: we can just start with the spins totally randomised. Then, what we want is some way to
transform our naive system into one that represents a physical reality: in this case, one that exists with maximum probability.
Looking at the expression for the Boltzmann distribution above, there is no reason that we can't just differentiate it with respect
to all the spins of the system, to see how the probability would change if that spin were to change. In a line, we calculate this
derivative:

$$ Z \cdot \nabla p(\textbf{s}) = \sum_i{\beta (H + J \sum_{i \subset n_i}{s_i}) \cdot p(\textbf{s}) \mathbf{\hat{s_i}}}$$

I changed from working over a set of spins, $$\{s_i\}$$ to a vector to reflect that we are working in a vector space where we want
to push and pull spin values to see what gives us the best probability. The sum over the *i*s within this $$n_i$$ means all the
spins that are nearest-neighbour to *i*. I think this calculation has a pleasing result, because it sort of shows us that the
probability of a state gets better in the same direction as the local 'force' working on each spin: if we move in the direction
to line the spin up with the external field as well as its neighbours, the probability of the state increases.

What we also see is that this local force is mediated by the inverse temperature: if $$\beta$$ decreases (the system gets hotter)
then the force moving us to a more probable state reduces, and we stay in our initial configuration (more on this in the next
section).

### Thermal Fluctuations and turning this into code

First, a quick note: all the code that this is based on can be found on my GitHub, here: <https://github.com/whatf0xx/pysing/>.
My apologies to anyone who wanted to do something better with the `pysing` repository name.

The maths above shows us how we can work out a better state for individual spins based on maximising the probability of the
over all distribution. What we want is to be able to follow the path that this lays for us in phase space (i.e. where directions
are given by the states of each spin in the system). It's important to remember that spins can **only** have the value +/- 1, so
a really large gradient should still give us a value in the end of 1, and a really large negative gradient should give us a value
of -1.

```
def probability_gradient(self) -> np.ndarray:
        """
        Calculate the (not normalized) gradient of the probability with respect
        to the spins of the system. the gradient is taken over the product of
        the probability of the microstate and the partition function (see the
        `z_prob` property), so its really the gradient of the numerator of the
        Boltzmann distribution. There should also be a factor of the probability
        of the microstate, but this is removed for now for numerical simplicity
        as this should affect all the spins the same, anyway.
        """
        beta = self.inverse_temp
        H = self.H
        J = self.J
        # p = self.z_prob
        return np.array(
            [beta * (H + J * spin.nn_sum()) for spin in self.spins]
        )
```

Note above the commented out probability for the microstate, *p*. In the end, this just scales the gradient for every spin in the
system by the same amount, so it doesn't really add anything.

We also have to work out how to treat the thermal term ($$\beta$$) in our calculation: this should pull us back to randomness (if
that's where we started) or ideally push us *towards* it if we occupy a highly structured microstate, too. This is basically
achieved by introducing a degree of randomness to the system: we move from microstate to microstate, and calculate the spin
configuration of the new system based on a stochastic evolution from the one before. To calculate the chances of being spin up or
down, we use the gradient calculated above, so very high gradients lead to spin up with probability approaching one, and very
low gradients lead to spin down with probability approaching one. If the temperature is high, the gradient goes to zero, and the
chances of getting either spin moves towards half. This mapping is achieved using a shifted hyperbolic tangent function, which
isn't so different to what might get used as an activation in a machine learning neurone:

```
def evolution_probs(self) -> np.ndarray:
        """
        Calculate the probability for transforming to spin up (1.) when the
        model evolves.
        """
        return (1 + np.tanh(self.probability_gradient)) / 2
```

Finally, we want to evolve the system, taking the probability for each spin's new configuration to be this probability. There is
one caveat which I noticed for small systems: under quite a lot of conditions, in zero external field, the whole system quickly
approaches a checkerboard configuration. This happens because every single spin is surrounded by opposite nearest neighbours, 
leading all of them to flip indefinitely, but never to align with its surroundings. To get around this, I introduced a parameter
to only update a fraction of the system's spins everytime. This would mean that if we got to a checkerboard configuration, then
the spins wouldn't all flip at once, getting us back to the physically sensible microstate where they line up, rather than bouncing
around each other for the rest of time:

```
def evolve(self):
        """
        According to the calculated probability gradient, let the system evolve
        into a more likely state.
        """
        for spin, p in zip(self.spins, self.evolution_probs):
            if random() > 0.2:
                # don't update all spins to avoid back-and-forth flipping
                spin.value = choices([-1., 1.], weights=[1-p, p], k=1)[0]
```

### Playing around

All that was left was to play around with some parameters and work out conditions for the system to yield sensible results.
The system is defined in terms of 3 variables, *(T, H, J)*, all described above. What we are mostly interested in is behaviour
surrounding the system's critical temperature, $$T_c$$, which, in the case of the 2D-Ising model, is defined entirely by *J*,
according to [an analytic solution due to Onsager](https://en.wikipedia.org/wiki/Ising_model#Onsager's_exact_solution). This
meant that I could set *T* so that $$\beta$$ would be about 1 (to keep calculations simple) and then adjust *J* to match the
critical temperature (or critical coupling, if you like) and hence achieve a critical system. Because the system was critical,
only small values of *H* would be needed to achieve high degrees of magnetisation in the system (around the critical temperature,
the magnetic susceptibility also becomes very large).

The resulting simulation looks something like this for a few worthwhile values of the system parameters:

![My Ising model]({{ site.url }}{{ site.baseurl }}/assets/images/ising.png)

### Closing Remarks

There are a couple of things that I was really happy about with how this simulation worked out.

When I was calculating the gradient of the Boltzmann distribution initially, I kept getting terms that were proportional to the
whole system's magnetisation, or the sum over nearest-neighbours for the whole system. This was frustrating, because in the end
the spins don't care about the whole system, they only care about their immediate surroundings. Indeed, even in the final gradient
calculation there is still a term of $$p(\mathbf{s})$$, but at least we can factor that out without changing anything in our results.
Otherwise, though, I was happy to see that we do get this simple, descriptive term of $$\beta(H + J\sum_{i \subset n_i}{s_i})$$
which encapsulates the dynamics we are trying to describe, and nothing else, really.

I was also happy with how the simulation was designed, in particular, for each spin I stored its nearest neighbours in a hash map,
so that they could be retrieved quickly on each iteration. I wonder, though, if this was really the fastest way to go and whether
some clever array manipulation could have sped things up.

I guess now I should get off and check out this 'Metropolis algorithm' that's on the Ising model's Wikipedia page. Maybe I will write
another post comparing what I came up with to a sensible solution.
