---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
title: Flatten-the-Curve Sim
nav_order: 1
---

# Flatten-the-Curve Simulation

I read [this article](https://www.washingtonpost.com/graphics/2020/world/corona-simulator/) with a social distancing simulator on Washington Post a few weeks back (I don't remember who shared it, but whoever it was thank you!). I thought it would be fun to recreate using vanilla JavaScript!

I put together a first draft that you can find [here](https://mjumbewu.github.io/flatten-curve-sim/firstdraft/step20.html). I also found the [Simulating an epidemic](https://www.youtube.com/watch?v=gxAaO2rsdIs) video from 3Blue1Brown very helpful in thinking about approaches to the model. You can see the code for the steps I went through [in this GitHub repository](https://github.com/mjumbewu/flatten-curve-sim/tree/master/firstdraft). My goal with _this_ site that you're reading now is to clean up the code and take you through the steps that it takes to create a simulation like this in JavaScript.

## Why?

Simulation is a helpful tool for exploring the variables that affect a phenomenon. As has been mentioned in many places in reference to the spread of COVID-19, humans have a poor intuitive understanding of exponential growth. I think there are many things about a pandemic that we have a pretty weak intuitive grasp on. Evolution has not prepared us well for comprehending the complexity of factors involved in something that happens so quickly to such a large portion of the planet.

Simulation allows us to describe aspects of the world as we understand it and see what happens to the world as we let those aspects play out over time. In doing so we can see how things that we think of as simple local rules of interaction can have far-reaching effects. Then, changing the parameters of our model allows us to ask what-would-happen-if questions of that world. In the specific case of a model of people within the context of a pandemic, hopefully our simulation can help us get a better understanding of what can happen under situations like:
- medical infrastructure with a finite capacity
- varying degrees of respect for stay-home orders
- timing around relaxing of mobility restrictions

That said, what we're building here isn't necessarily the best model for COVID-19 transmission. It’s not going to necessarily be the thing that allows you to internalize the enormity of what’s going on. It's not so much that I think _this_ simulator is most important. Rather, I just want to introduce how one might approach building _a_ simulator … in JavaScript.

Moreover, as I mentioned above, I enjoy creating models and simulations. I think it's fun, and I think it would be fun to share the technique with others.

## How?

I'm going to do this with JavaScript. It's a decent little language. And I'm not going to use any frameworks -- just the tools that most _modern_ browsers give us. It is not my goal to make this work in older browsers versions. Instead, I'll use whatever standard JS features work across the latest versions of browsers like Chrome and Firefox (and probably Edge and Safari). I'm interested in showing how a simulation like the Washington Post one is constructed and works in JavaScript. How to construct code for cross-browser compatibility is generally really important to understand, it just isn't the point of this series.

To follow along, some prior knowledge will be helpful (though I'll also try to be as explanatory as possible when I remember to be, so don't get discouraged):
* JavaScript -- but pretty much any programming knowledge and a willingness to get your hands dirty with some searching on Google and Stack Overflow will suffice.
* Some familiarity with trigonometric functions (sine, cosine, tangent) and some basic geometry will help you to understand the physical simulation part.
* Basic stats for the charting stuff -- basically understanding what averages are, and maybe knowing how to read a box-and-whisker plot (if I get that far).

Again, I will try to be clear about anything that I feel is not self-explanatory or easily Google-able. If an explanation is out of the scope of this project, or beyond my expertise, I'll try to remember to call that out as well.

Lastly, I'm going to be explaining a number of things that I am far from an expert in. Sometimes I'll choose one technique over another because it's easier to explain or more comprehensible to learners. Other times I may choose one technique over another because I don't know any better. There's so much I either never learned or have learned and since forgotten about epidemiology, statistics, vector transformations, and even JavaScript. Suggestions and corrections are welcome -- let's say in the [GitHub Issues](https://github.com/mjumbewu/flatten-curve-sim-narrative/issues) for now.

## Rough Outline

Done so far...
- [**Initial Setup**](initial-setup) -- Create an HTML document structure for the interface.
- [**The Data Model**](data-model) -- Establish the data model (the types of objects, their general properties, and their relationships) that we'll use for the simulation.
- [**Drawing the World**](drawing-the-world) -- Create methods to draw the data model to the UI.
- [**Introducing Time**](introducing-time) -- Update the state of the simulation agents as time ticks on.
- [**Colliding with Barriers**](interacting-with-boundaries) -- Getting the agents to stay within the boundaries of the world.
- [**Colliding with Other Agents**](interacting-with-agents) -- Making agents aware of when they come into contact with each other, and having them respond to those interactions.

Still to come...
- **Introducing Infection** -- Adding the concept of infection to the model and simulation.
- **Charting Stats About the World** -- Adding in chart-based visualizations of the aggregate properties of the world's agents.

----------

Ok, let's get started!

[The Initial Setup](initial-setup){: .btn }