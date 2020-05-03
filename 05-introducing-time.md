---
layout: page
title: Introducing Time
permalink: introducing-time
nav_order: 5
---

# Introducing Time

This simulation isn't very exciting right now. One of the biggest reasons is that it's not dynamic; there's no movement, no change over time, because there is no time. All we have right now is the ability to build a static state of a world, and to display that static state.

## Agent Time

To manage time, we will add a `step` method to anything that will change in the simulation, starting with the `Agent`s. Each time `step` is called, the `Agent` will step forward in time by some amount (`Δt`, where delta (Δ) is often used to denote the amount of change in some property; also I thought it would be fun to use a non-ascii character in these variable names). We can use the agent's `direction` and `speed` to calculate the agent's new `x` and `y` coordinates. We use `cos` to get the change in the horizontal direction, and `sin` to get the change in the vertical direction ([sohcahtoa, sohcahtoa, sohcahtoa](https://www.mathsisfun.com/algebra/sohcahtoa.html)).

In the _model.agent.js_ module, add the following before the `export` statement:

```js
Agent.prototype.step = function(Δt=1) {
  const Δx = Math.cos(this.direction) * this.speed * Δt
  const Δy = Math.sin(this.direction) * this.speed * Δt

  const x = this.x + Δx
  const y = this.y + Δy

  const time = this.time + Δt

  return new Agent({
    ...this,
    x, y,
    time,
  })
}
```

A couple things to note about the above:
* By default we step forward by 1 time unit (`Δt=1`).
* Instead of modifying the `Agent` that we call `step` on, we're going to create and return a copy of the agent with new parameters. As a rule, we are going to avoid modifying properties on the `Agent` and `World` objects after they are created, preferring instead to treat them as immutable (this means that our browsers' garbage collectors are going to have their work cut out for them, but I think they're up to the task).

Also, we need to initialize the `Agent` instance's `time`. In the constructor function, add the following:

```js
this.time = params.time || 0
```

## World Time

Next we'll add a `step` method to the `World` that will mainly exist to step all of the world's component parts at the same time. In the _model.world.js_ module, add the following before the `export` statement:

```js
World.prototype.step = function(Δt=1) {
  const agents = this.agents.map(a => a.step(Δt))
  const time = this.time + Δt

  return new World({
    ...this,
    agents,
    time,
  })
}
```

Note again that we create and return a new `World` instead of modifying the existing one. Also note that we do not need a `step` method on our `Boundary` objects because, for now, our boundaries won't change over time (though we may decide to introduce that later, e.g. to explore quarantine scenarios).

Also, we need to initialize the `World` instance's `time`. In the constructor function, add the following:

```js
this.time = params.time || 0
```

## Moving Time Forward

Now if we call `world.step()` in our _main.js_ module we will get a new `World` instance stepped one time unit into the future. We can use our `simview.draw()` method to update our SVG drawing with the state of the new world as well. We could also create a `for` loop to do this a bunch of times in a row for us. However, because of the single-threaded nature of how JS runs in the browser, we'll need to set timeouts between each world step in order to have the UI update and our simulation look animated (see [this article](https://dev.to/lydiahallie/javascript-visualized-event-loop-3dif) for an introduction to the way that JS event loop schedules things to happen, or [this simulator](http://latentflip.com/loupe/) and the associated [JSConf Talk](https://www.youtube.com/watch?v=8aGhZQkoFbQ) to visualize how the scheduling works).

To manage the process of driving the `World` model forward, let's create a new type of `Simulation` object that we'll use to `start` and `stop` time in our simulated world. Since the `Simulation` is going to schedule each step of our world using the JS event loop, let's also allow it to use events to let other parts of our program know when a step has been completed.

Create a _simulation.js_ module:

_simulation.js:_
```js
class Simulation {
  constructor(params) {
    this.world = params.world
    this.stepDelay = params.stepDelay || 0

    // Create an EventTarget for the simulation that will allow us to notify
    // other parts of the program when simulation events happen, like when
    // we've started or stopped running, or when we've completed a step.
    this.target = new EventTarget()
  }

  start() {
    // Schedule the first step with no delay.
    this._scheduleNextSteps(0)

    // Trigger a "start" event to let the rest of the program know that we've
    // started.
    this.trigger('start')

    return this
  }

  _scheduleNextSteps(delay) {
    // After a delay of some milliseconds, queue the _runNextStep method to be
    // called again.
    this.stepTimer = setTimeout(() => this._runNextStep(), delay)
  }

  _runNextStep(keepGoing=true) {
    // Step the world forward.
    const world0 = this.world
    this.world = world0.step()

    // Schedule the next step, if we're supposed to.
    if (keepGoing) { this._scheduleNextSteps(this.stepDelay) }

    // Trigger a "step" event to let the rest of the program know that we've
    // completed another step.
    this.trigger('step')
  }

  step() {
    // Sometimes we might want to just perform a single step.
    this._runNextStep(keepGoing=false)
    return this
  }

  stop() {
    // To stop the simulation, cancel any upcoming scheduled steps.
    clearTimeout(this.stepTimer)
    delete this.stepTimer

    // trigger a "stop" event to let the rest of the program know that we've
    // stopped the simulation.
    this.trigger('stop')

    return this
  }

  trigger(eventName) {
    this.target.dispatchEvent(new CustomEvent(eventName, {
      detail: { sim: this }
    }))
    return this
  }

  on(eventName, callback) {
    this.target.addEventListener(eventName, callback)
    return this
  }
}

export { Simulation }
```

Now let's modify our _main.js_ module to use this new simulation driver. In the imports in _main.js_, add:

```js
import { Simulation } from './simulation.js'
```

Then, at the very bottom of the module, add:

```js
let sim = new Simulation({world})
sim.on('step', e => simview.draw(e.detail.sim.world))
sim.start()
```

The second line there creates a callback that redraws our `<svg>` image whenever the `sim` emits a `step` event. The last line starts the simulation. At this point, when you refresh the page, you should see the dot that we created in [Drawing the World](drawing-the-world) shoot off in a random direction.

## More Motion!

Just because one agent doesn't make for a very exciting simulation, let's add more agents &mdash; say, 100, randomly placed. In your _main.js_ module, replace everything down to the definition of `agents` with the following:

```js
import { Agent } from './model.agent.js'
import { Boundary } from './model.boundary.js'
import { World } from './model.world.js'
import { Simulation } from './simulation.js'
import { SVGSimView } from './view.svgsim.js'

let el = document.querySelector('#simulation-world')
const W = el.viewBox.baseVal.width
const H = el.viewBox.baseVal.height
const R = 5
const π = Math.PI

function rand() { return Math.random() }
function randBetween(lower, upper) { return rand() * (upper - lower) + lower }

let agents = Array(100)
  .fill(null)
  .map(() =>
    new Agent({
      x: randBetween(R, W - R),
      y: randBetween(R, H - R),
      radius: R,
      direction: rand() * 2*π,
      speed: 1,
    })
  )
```