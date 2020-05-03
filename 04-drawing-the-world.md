---
layout: page
title: Drawing the World
permalink: drawing-the-world
nav_order: 4
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

Here we used a `class` with its own `constructor` method, instead of a constructor function as we did in the model modules. The difference is purely syntactic, but I felt that a constructor function would better suit the models. I think of the models as being more about data than about the behavior attached to that data, and the functional syntax seemed closer to the raw structure than the scaffolding of a class would. However, I retain the right to change my mind.

Next, let's create a basic simulation world with a single agent and four walls, just to have something to draw. In the _main.js_ module, enter the following:

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
  new Boundary({x1: 0, y1: 0, x2: W, y2: 0}),
  new Boundary({x1: W, y1: 0, x2: W, y2: H}),
  new Boundary({x1: W, y1: H, x2: 0, y2: H}),
  new Boundary({x1: 0, y1: H, x2: 0, y2: 0}),
]

let world = new World({agents, boundaries})

let simview = new SVGSimView({ el })
simview.draw(world)
```

Now our module calls a `draw` method on the `sim` view object, but we haven't implemented this method yet. What we want the `draw` method to do is sync up the `agents` and `boundaries` in our world to SVG elements in our simulation image. In the `SVGSimView` class, let's insert the following method:

```js
draw(world) {
  // CLEAR THE DRAWING
  // -----------------
  // First, empty the drawing my removing any children of the <svg> element.
  // Initially this element is empty (see your "index.html" file), so there
  // won't be anything to remove, but on subsequent runs of this function, we
  // may have children. You'll see how those children get created below.
  for (let child of this.worldEl.children) {
    child.remove()
  }

  // DRAW THE AGENTS
  // ---------------
  // Loop through each one of the agents in the world, which is passed in as
  // an argument to this function, and for each one create a <circle> element.
  // Add each circle to the <svg> element, and set the circle attributes to
  // reflect the position and size of the agent.
  for (let agent of world.agents) {
    let agentEl = document.createElementNS('http://www.w3.org/2000/svg', 'circle')
    agentEl.setAttribute('class', 'simulation-agent')
    this.worldEl.appendChild(agentEl)

    agentEl.setAttribute('cx', agent.x)
    agentEl.setAttribute('cy', agent.y)
    agentEl.setAttribute('r', agent.radius)
  }

  // DRAW THE BOUNDARIES
  // -------------------
  // Similar to the agents, loop through each of the boundaries in the world.
  // Add a line to represent the boundary.
  for (let boundary of world.boundaries) {
    let boundaryEl = document.createElementNS('http://www.w3.org/2000/svg', 'line')
    boundaryEl.setAttribute('class', 'simulation-boundary')
    this.worldEl.appendChild(boundaryEl)

    boundaryEl.setAttribute('x1', boundary.x1)
    boundaryEl.setAttribute('y1', boundary.y1)
    boundaryEl.setAttribute('x2', boundary.x2)
    boundaryEl.setAttribute('y2', boundary.y2)
  }
}
```

Finally let's add some very basic styling for the simulation element. Create a _simulation.css_ file and link to it from the _index.html_ file:

_simulation.css:_
```css
/*== The UI Layout */
body {
  display: flex;
  margin: 0;
  padding: 0;
  height: 100vh;
  align-items: center;
}

#simulation-section {
  flex-grow: 1;
}

#simulation-world {
  max-height: 95vh;
}

/*== The Simulation */
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

At this point, if we load the page in a browser we should see a dot surrounded by a box. Great! I mean, it's not very exciting, but it's a great start. In the next step we'll add in a few more agents and introduce the concept of time into our simulation. But first...

## A Bit of Optimization (optional)

This bit is optional. The draw function will essentially have the same end-result either way. If you would like to skip straight to the next step, [feel free](introducing-time).

Notice that our `draw` method starts by clearing out all of the elements it contains and then recreates all of those elements again. This feels like it will be really inefficient, so instead let's have the view keep track of these SVG elements. One way that we could do this is to set the element as an attribute on its corresponding agent/boundary object (e.g., `agent.el = agentEl`). However, we want to keep the view and the model classes decoupled, so instead we're going to borrow the notion of "zipping" from Python (I don't think JS has any kind of zip function built in, but if it does, someone please [let me know](https://github.com/mjumbewu/flatten-curve-sim-narrative/issues)). When we zip two arrays, we end up with a new array that contains corresponding pairs from the source arrays. For example, say we have these two arrays:

```js
const array1 = [1, 2, 3]
const array2 = ['a', 'b', 'c']
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

In our case we'll want the latter. Let's implement the `zipLongest` function in a new _utils.js_ module:

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
      boundaryEl.setAttribute('x1', boundary.x1)
      boundaryEl.setAttribute('y1', boundary.y1)
      boundaryEl.setAttribute('x2', boundary.x2)
      boundaryEl.setAttribute('y2', boundary.y2)
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

Great, that should be a little better. With the number of times that we'll end up calling this `draw` method each second, it will likely be worthwhile for it to be as efficient as reasonable.

----------

Now that we have a drawn world, let's introduce the concept of time!

[Introducing Time](introducing-time){: .btn }