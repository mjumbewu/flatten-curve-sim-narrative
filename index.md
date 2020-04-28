---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
title: Flatten-the-Curve Sim
nav_order: 1
---

# Flatten-the-Curve Simulation

I saw [this article](https://www.washingtonpost.com/graphics/2020/world/corona-simulator/) with a social
distancing simulator on Washington Post (I don't remember who shared it, but whoever it was thank you!). I
thought it would be fun to recreate!

I put together a first draft that you can find [here](https://mjumbewu.github.io/flatten-curve-sim/firstdraft/step20.html). You can see the code for the steps I went through [in this GitHub repository](https://github.com/mjumbewu/flatten-curve-sim/tree/master/firstdraft). My goal with _this_ site that you're reading now is to clean up the code and take you through the steps that it takes to create a simulation like this in JavaScript.

## Why?

Simulation is a helpful tool for exploring the variables that affect a phenomenon. As has been mentioned in many places in reference to the spread of COVID-19, humans have a poor intuitive understanding of exponential growth. I think there are many things about a pandemic that we have a pretty weak intuitive grasp on. Evolution has not prepared us well for comprehending the complexity of factors involved in something that happens so quickly to such a large portion of the planet.

That said, what I’m building here isn't necessarily THEE ONE AND ONLY BEST model for COVID-19 transmission. It’s not going to necessarily be the thing that allows you to internalize the enormity of what’s going on. It's not so much that I think _this_ simulator is most important. Rather, I just want to introduce how one might approach building a simulator … in JavaScript.

Moreover, as I mentioned above, I enjoy creating models and simulations. I think it's fun, and I think it would be fun to share the technique with others.

## How?

I'm going to do this with JavaScript. It's a decent little language. And I'm not going to use any frameworks -- just the tools that most _modern_ browsers give us. It is not my goal to make this work in older browsers versions. Instead, I'll use whatever standard JS features work across the latest versions of browsers like Chrome and Firefox (and probably Edge and Safari). I'm interested in showing how a simulation like the Washington Post one is constructed and works in JavaScript. How to construct code for cross-browser compatibility is generally really important to understand, it just isn't the point of this series.

To follow along, some prior knowledge will be helpful (though I'll also try to be as explanatory as possible when I remember to be, so don't get discouraged):
* JavaScript -- but pretty much any programming knowledge and a willingness to get your hands dirty with some searching on Google and Stack Overflow will suffice.
* Some familiarity with trigonometric functions (sine, cosine, tangent) and some basic geometry will help you to understand the physical simulation part.
* Basic stats for the charting stuff -- basically understanding what averages are, and maybe knowing how to read a box-and-whisker plot (if I get that far).

Again, I will try to be clear about anything that I feel is not self-explanatory or easily Google-able. If an explanation is out of the scope of this project, or beyond my expertise, I'll try to remember to call that out as well.

Lastly, I'm going to be explaining a number of things that I am far from an expert in. Sometimes I'll choose one technique over another because it's easier to explain or more comprehensible to learners. Other times I may choose one technique over another because I don't know any better. There's so much I either never learned or have learned and since forgotten about epidemiology, statistics, vector transformations, and even JavaScript. Suggestions and corrections are welcome -- let's say in the [GitHub Issues](https://github.com/mjumbewu/flatten-curve-sim-narrative/issues) for now.

Ok, let's get started!

[The Initial Setup](initial-setup){: .btn }