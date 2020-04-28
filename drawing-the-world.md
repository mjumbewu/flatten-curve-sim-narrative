---
layout: page
title: The Data Model
---

# Drawing the World

Now we're going to create a class that we'll use to draw a simulation world into an SVG image. We'll start with a placeholder class in a new module:

_view.svgsim.js:_
```js
class SVGSimView {
  constructor(options) {
    this.worldEl = options.el
  }
}
```

Then let's create a basic simulation world with a single agent and four walls, just to have something to draw. In the _main.js_ module, enter the following:

```js
import { Agent } from './model.agent.js'
import { Boundary } from './model.boundary.js'
import { World } from './model.world.js'
import { SVGSimView } from './view.svgsim.js'

let el = document.querySelector('#simulation-world')
const W = el.viewBox.baseVal.width
const H = el.viewBox.baseVal.height

let agents = [
  new Agent({x: W / 2, y: H / 2, radius: 5})
]

let boundaries = [
  new Boundary({x0: 0, y0: 0, x1: W, y1: 0}),
  new Boundary({x0: W, y0: 0, x1: W, y1: H}),
  new Boundary({x0: W, y0: H, x1: 0, y1: H}),
  new Boundary({x0: 0, y0: H, x1: 0, y1: 0}),
]

let world = new World({agents, boundaries})

let sim = new SVGSimView({ el })
sim.draw(world)
```

Now our module calls a `draw` method on the `sim` view object, but we haven't implemented this method yet. What we want the `draw` method to do is sync up the `agents` and `boundaries` in our world to SVG elements in our simulation image. In the `SVGSimView` class, let's insert the following method:

```js
draw(world) {
  // == EMPTY THE WORLD ==
  for (let child of this.worldEl.children) {
    child.remove()
  }

  // == DRAW THE AGENTS ==
  for (let agent of world.agents) {
    let agentEl = document.createElementNS('http://www.w3.org/2000/svg', 'circle')
    agentEl.setAttribute('class', 'simulation-agent')
    this.worldEl.appendChild(agentEl)

    agentEl.setAttribute('cx', agent.x)
    agentEl.setAttribute('cy', agent.y)
    agentEl.setAttribute('r', agent.radius)
  }

  // == DRAW THE BOUNDARIES ==
  for (let boundary of world.boundaries) {
    let boundaryEl = document.createElementNS('http://www.w3.org/2000/svg', 'line')
    boundaryEl.setAttribute('class', 'simulation-boundary')
    this.worldEl.appendChild(boundaryEl)

    boundaryEl.setAttribute('x1', boundary.x0)
    boundaryEl.setAttribute('y1', boundary.y0)
    boundaryEl.setAttribute('x2', boundary.x1)
    boundaryEl.setAttribute('y2', boundary.y1)
  }
}
```

## A Bit of Optimization

Notice that our `draw` method starts by clearing out all of the elements it contains and then recreates all of those elements again. This feels like it will be really inefficient, so instead let's have the view keep track of these SVG elements. So that we can keep the view and the model classes decoupled, we're going to borrow the notion of "zipping" from Python (I don't think JS has any kind of zip function built in, but if it does, someone please [let me know](https://github.com/mjumbewu/flatten-curve-sim-narrative/issues)). When we zip two arrays, we end up with a new array that contains corresponding pairs from the source arrays. For example, say we have these two arrays:

```js
array1 = [1, 2, 3]
array2 = ['a', 'b', 'c']
```

Zipping these two arrays together would result in the following array:

```js
zip(array1, array2)
// [ [1, 'a'], [2, 'b'], [3, 'c'] ]
```

If the arrays are different lengths, we could either (a) return a new array as long as the shortest, ensuring that we have enough items from both lists to contribute to the pairs, or (b) return a new array as long as the longest array, inserting some default value when we run out of elements from the shorter array.

```js
zipShortest([1, 2, 3, 4, 5], ['a', 'b', 'c'])
// [ [1, 'a'], [2, 'b'], [3, 'c'] ]

zipLongest([1, 2, 3, 4, 5], ['a', 'b', 'c'])
// [ [1, 'a'], [2, 'b'], [3, 'c'], [4, undefined], [5, undefined] ]
```

Let's implement the `zipLongest` function in a _utils.js_ module:

_utils.js:_
```js
function* zipLongest(array1, array2) {
  const maxLength = Math.max(array1.length, array2.length)
  for (let i = 0; i < maxLength; ++i) {
    yield [array1[i], array2[i]]
  }
}

export { zipLongest }
```

Then we can use this function from within our `draw` method. Let's replace our _view.svgsim.js_ module with the following:

```js
import { zipLongest } from './utils.js'

class SVGSimView {
  constructor(options) {
    this.worldEl = options.el
    this.agentEls = []
    this.boundaryEls = []
  }

  draw(world) {
    // == DRAW THE AGENTS ==
    // Match up each agent in the world with an SVG circle element. Ensure that
    // we're dealing with exactly as many circle elements as we are agents. If
    // there are more agents than circles, create new circle elements for the
    // unrepresented agents. If there are more circles than agents, remove the
    // extra elements.
    for (let [agent, agentEl] of zipLongest(world.agents, this.agentEls)) {
      if (!agent) {
        if (agentEl) agentEl.remove()
        continue
      }

      agentEl = agentEl || this.makeAgentEl()
      agentEl.setAttribute('cx', agent.x)
      agentEl.setAttribute('cy', agent.y)
      agentEl.setAttribute('r', agent.radius)
    }

    // == DRAW THE BOUNDARIES ==
    // Next, do the same kind of matching between world boundaries and SVG
    // line elements.
    for (let [boundary, boundaryEl] of zipLongest(world.boundaries, this.boundaryEls)) {
      if (!boundary) {
        if (boundaryEl) boundaryEl.remove()
        continue
      }

      boundaryEl = boundaryEl || this.makeBoundaryEl()
      boundaryEl.setAttribute('x1', boundary.x0)
      boundaryEl.setAttribute('y1', boundary.y0)
      boundaryEl.setAttribute('x2', boundary.x1)
      boundaryEl.setAttribute('y2', boundary.y1)
    }
  }

  makeAgentEl() {
    let agentEl = document.createElementNS('http://www.w3.org/2000/svg', 'circle')
    agentEl.setAttribute('class', 'simulation-agent')
    this.agentEls.push(agentEl)
    this.worldEl.appendChild(agentEl)
    return agentEl
  }

  makeBoundaryEl() {
    let boundaryEl = document.createElementNS('http://www.w3.org/2000/svg', 'line')
    boundaryEl.setAttribute('class', 'simulation-boundary')
    this.boundaryEls.push(boundaryEl)
    this.worldEl.appendChild(boundaryEl)
    return boundaryEl
  }
}

export {
  SVGSimView
}
```

Finally let's add some very basic styling for the simulation element. Create a _simulation.css_ file and link to it from the _index.html_ file:

_simulation.css:_
```css
body {
  margin: 0;
  padding: 0;
}

#simulation-world {
  max-width: 100%;
  max-height: 100vh;
}

.simulation-boundary {
  stroke: gray;
  stroke-width: 5px;
}
```

_index.html:_
```html
<!DOCTYPE html>

<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Flatten-The-Curve Simulation</title>
  <link rel="stylesheet" href="simulation.css">
</head>
...
```

At this point, if we load the page in a browser we should see a dot surrounded by a box. Great! I mean, it's not very exciting, but it's a great start. Next, let's add in a few more agents and introduce the concept of time into our simulation.

...