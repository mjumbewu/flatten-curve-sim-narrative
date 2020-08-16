---
layout: page
title: Introducing Infection
permalink: introducing-infection
nav_order: 8
---

# Introducing Infection

We're going to use a state machine to track the progress of any given agent's infection.

{% include illustrations/infection_state_machine.html %}

To choose my colors I went over to [Paletton](https://paletton.com/#uid=7000u0kllllaFw0g0qFqFg0w0aF) and picked a pretty default tetrad. Since I have 5 states and need 5 colors, I'm going to throw in gray for deceased.

## Setting Infectiousness

Every time one infected agent collides with another uninfected agent, there's a chance that the first will pass on the infection to the second. There are a number of ways that we could determine the likelihood of infection, but I choose to use the following:

You may have heard of an `R₀` (or "R-naught"). In the context of a model like this you could think of `R₀` this way: If an infected agent has an unlimited supply of uninfected agents around it, `R₀` is the number of agents that the agent will most likely infect while it is contagious.

a.infectiousness = TARGET_R0 / (COLLISIONS_PER_STEP * INFECTIOUS_STEPS)
