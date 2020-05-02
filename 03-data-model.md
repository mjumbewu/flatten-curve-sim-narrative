---
layout: page
title: The Data Model
permalink: data-model
nav_order: 3
---

# The Data Model

> All models are approximations. Assumptions, whether implied or clearly stated, are never exactly true. _All models are wrong, but some models are useful_. So the question you need to ask is not "Is the model true?" (it never is) but "Is the model good enough for this particular application?"
>
> &mdash; George Box

The _On the Media_ podcast has a pretty good [interview with Joshua Epstein](https://pca.st/episode/9ba60afd-a125-42b5-a69d-3a6d973e2c78?t=1918) on models in the time of COVID-19 ("Model Behavior", _On the Media_, 2020 April 17). I recommend it.

Let's describe a simple model to start with. We can complicate this model as we progress. The specific type of model we are going to build is an [agent-based model](https://en.wikipedia.org/wiki/Agent-based_model) (ABM). Generally, an ABM consists of many virtual "agents" that each represent some real-world individual or collective. These agents act and interact within a virtual world. Often, these agents are all similar to each other, but differ in some randomized ways. In spite of this randomness, over time, their interactions result in some consistent patterns.

Let's say each agent in our model represents one person. Further let's say that each of those persons is positioned at some point `[x, y]` and moves around within the world in some direction with some speed. The world itself has edges, so that all the people move around in the same space. This space could represent a neighborhood, a city, a country -- use your imagination. The edges represent the borders -- physical, economic, or social -- that confine people to that space.

Now we can start coding this simple data model. Let's create a separate [JS module](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) for each entity:

_model.agent.js:_
```js
function Agent(params) {
  this.x = params.x
  this.y = params.y
  this.radius = params.radius
  this.direction = params.direction
  this.speed = params.speed
}

export { Agent }
```

_model.boundary.js:_
```js
function Boundary(params) {
  this.x1 = params.x1
  this.y1 = params.y1
  this.x2 = params.x2
  this.y2 = params.y2
}

export { Boundary }
```

_model.world.js:_
```js
function World(params) {
  this.agents = params.agents
  this.boundaries = params.boundaries
}

export { World }
```

Then, let's create a module to tie them all together.

_main.js:_
```js
import { Agent } from './model.agent.js'
import { Boundary } from './model.boundary.js'
import { World } from './model.world.js'
```

Finally, let's add a `script` tag to the bottom of our page to import the main module.

```html
  ...
  <section id="summary-section">
  </section>

  <script type="module" src="main.js"></script>
</body>
</html>
```

[Drawing the World](drawing-the-world){: .btn }
