---
layout: page
title: Interacting With Boundaries
permalink: interacting-with-boundaries
nav_order: 6
---

# Interacting With Boundaries

We're going to start formalizing a few geometric concepts in our code. Specifically we're going to be using:

- `Point` - So that we're dealing with our **x** and **y** coordinates in a consistent way, we're going to package them into point structures. All out points are going to be two-dimensional.
- `Segment` - The portion of a line between two points. It's described by two points.
- `Vector` - If you've never worked with mathematical vectors before, you can think of them as an arrow that goes in a certain **direction** for a certain distance (**magnitude**). Alternatively, they're like the arrow (or the displacement or offset) between two points. It's different than a segment, because a segment is fixed in space, whereas the same vector might be used to describe the offset between two different sets of points.

We'll take each of these three concepts one-by-one.

## Vector

Let's create a new module called _geometry.js_ for these concepts. The first thing we'll add to the module is the definition of a `Vector`.

We will most often define a vector using its x and y components, so those are what we will use as constructor arguments, and set on the object. We will want to do various types of vector arithmetic as well. For now, we will add `plus`, `minus`, and `times`:

_geometry.js_:
```js
class Vector {
  constructor(Δx, Δy) {
    this.Δx = Δx
    this.Δy = Δy
  }

  plus(other) {
    return new Vector(
      this.Δx + other.Δx,
      this.Δy + other.Δy,
    )
  }

  minus(other) {
    return new Vector(
      this.Δx - other.Δx,
      this.Δy - other.Δy,
    )
  }

  times(factor) {
    return new Vector(
      this.Δx * factor,
      this.Δy * factor,
    )
  }
}
```

## Point

Next we'll add the definition of a `Point`. A point is simple: it's just an x and y coordinate. Add the following to the module:

_geometry.js_:
```js
class Point {
  constructor(x, y) {
    this.x = x
    this.y = y
  }
}
```

## Segment

Finally, a segment is also relatively simple: it is the portion of a line between two points.

_geometry.js_:
```js
class Segment {
  constructor(p1, p2) {
    this.p1 = p1
    this.p2 = p2
  }
}
```

And let's not forget to export these classes so that we can use them elsewhere:

_geometry.js_:
```js
export { Point, Segment, Vector }
```

There's a fair bit we'll need to add to these classes, but for now we're ready to start making some agents bounce off the walls!

## Making an `Agent` Bounce

What we want to happen is, when an `Agent` hits a `Boundary`, the agent should bounce off of it.

Hm.

If you've ever done something like [instruct a robot to make a peanut butter and jelly sandwich](https://www.scientificamerican.com/article/robot-make-me-a-sandwich/), then you know that we haven't described what should happen in nearly enough detail yet. So we'll try again:

For a given `Agent` `a`, any time we call `a.step()`, `a` is moving in some direction with some speed. We'll call the `Vector` that could describe the agent's movement it's `velocity`, or `v` for short.

![An agent with its velocity vector](images/agent_bounce_01_velocity.jpg){: .illustration}

Most of the time, if there's nothing that's in agent `a`'s way, it should just be offset by the vector `v` over the course of a simulation step (that is, specifically, for a step that is 1 time unit long; generally for a step of `Δt` time units, the position of `a` should be offset by the vector `Δp = v .times (Δt)`).

![An agent offset after a step](images/agent_bounce_02_displacement.jpg){: .illustration}

However, sometimes there may be an obstacle in the way &mdash; specifically, in this case, a `Boundary`. We can tell whether a given boundary `b` is in the way by checking for collisions between `a` and the `b`. A collision will happen when the path that `a` takes during the course of it's step (which we can describe with a `Segment`) crosses the boundary `b` (which we can describe by another `Segment`).

![An agent colliding with a boundary](images/agent_bounce_03_collision.jpg){: .illustration}

Calculating whether two segments cross is definitely something defined well enough that we can get a computer to do it!

### Crossing Two Segments

Say we have to segments `s1` and `s2`. Each of these is a part of a line. Let's say the equation for the lines that these segments lie on are as follows:

```
y = m1 * x + b1
y = m2 * x + b2
```

Where `m1` and `m2` are the slopes and `b1` and `b2` are the y-intercepts (we'll deal with vertical lines, which have infinite slope no y-intercept, as a special case). If the two slopes are equal, then the lines are parallel and never intersect. Otherwise, we can find their intersection point by solving for `x` and `y`:

```
y = m1 * x + b1
y = m2 * x + b2

m1 * x + b1 = m2 * x + b2
m1 * x - m2 * x = b2 - b1
(m1 - m2) * x = b2 - b1

x = (b2 - b1) / (m1 - m2)
y = (m1 * x + b1) = (m2 * x + b2)
```

But remember, `x` and `y` are the coordinates of the point where the _lines_ cross. The segments only cross if that point is contained within both segments.

Let's add a getter called `slopeintercept` to the `Segment` class that will give us the slope and intercept of a segment's line.

```js
get slopeintercept() {
  const x1 = this.p1.x
  const y1 = this.p1.y
  const x2 = this.p2.x
  const y2 = this.p2.y

  const m = (y2 - y1) / (x2 - x1)
  const b = isFinite(m) ? y1 - m * x1 : null
  return [m, b]
}
```

Let's also add a method called `contains` to the `Segment` class that will tell us whether a given point lies along the segment's line and is between the segment's endpoints.

```js
contains(point) {
  const x1 = this.p1.x
  const y1 = this.p1.y
  const x2 = this.p2.x
  const y2 = this.p2.y

  const {x, y} = point
  const [m, b] = this.slopeintercept
  if (b === null && x !== x1) { return false }
  if (b !== null && y !== m * x + b) { return false}

  return (
    ( (x1 <= x && x <= x2) || (x1 >= x && x >= x2) ) &&
    ( (y1 <= y && y <= y2) || (y1 >= y && y >= y2) )
  )
}
```

Now we can add a `crosses` method to the `Segment` class that will return the point where two segments cross, or `null` if no such point exists (i.e., if the segments do not cross):

```js
crosses(other) {
  const s1 = this
  const s2 = other

  // Calculate the slopes. If they have the same slopes then they never
  // intersect. The one time that two segments might have the same slope but
  // still be parallel is if one is Infinity and the other is -Infinity, but
  // in that case both intercepts will be null.
  const [m1, b1] = s1.slopeintercept
  const [m2, b2] = s2.slopeintercept
  if (m1 === m2) { return null }
  if (b1 === null && b2 === null) { return null }

  // Calculate the intersection point (where y == m1 * x + b1 == m2 * x + b2).
  // Bear in mind in the code below that a null intercept implies a vertical
  // line with an infinite slope.
  const x = ( b1 === null ? s1.x1 : ( b2 === null ? s2.x1 : (b2 - b1) / (m1 - m2) ) )
  const y = ( b1 === null ? m2 * x + b2 : m1 * x + b1 )
  const i = new Point(x, y)

  // If and only if the intersection point is on both segments, it is valid
  if (s1.contains(i) && s2.contains(i)) { return i }
  else { return null }
}
```

### A Note About Real Numbers

Ok, now I'm going to save your future self some headache. If you already know why floating point arithmetic can be troublesome, you can just skip to the end of this section if you'd like. If you don't know what I'm talking about, then try out this instructive exercise.

Create a new HTML file in the same folder with the following contents:

_test_segments.html_:
```html
<html>
<body>
  <script type="module">
    import { Point, Segment } from './geometry.js'

    window.s1 = new Segment(
      new Point(-2, 0.3),
      new Point( 8, 0.3),
    )

    window.s2 = new Segment(
      new Point(-2, 0),
      new Point( 8, 10),
    )
  </script>
</body>
</html>
```

Here, we're creating two line segments. The first goes from `(x=-2, y=0.3)` to `(x=8, y=0.3)`. Since the y-coordinate on each end points is the same (`0.3`), it's a horizontal line. The second is a segment goes from `(x=-2, y=0)` to `(x=8, y=10)`. I did choose this particular segment for an illustrative reason, but there's nothing that makes it particularly special &mdash; it's just one of an infinite number of possible segments that crosses the first one.

Now, in your browser, open up http://localhost:8000/test_segments.html and open the JS console. Once there, run the following:

```js
s1.crosses(s2)
```

You should get a `Point` object with `(x=-1.7, y=0.3)`. Indeed, if you grab a pen and paper and do some calculations you will see that those values are correct. Next, in the JS console again, run the following:

```js
s2.crosses(s1)
```

If you do run that you'll see that the browser tells us `null`. &hellip; But certainly if two segments cross each other then they cross each other from either segment's perspective! Is that somehow not true?!

It is true &mdash; the logic is sound. The problem here is the numbers themselves. In the console, let's try to recreate what the `crosses` method is doing. First, let's get the slope and intercept for `s1`:

```js
[m1, b1] = s1.slopeintercept
```

We get `[ 0, 0.3 ]`. Seems correct. Then, get the slope and intercept for `s2`:

```js
[m2, b2] = s2.slopeintercept
```

Here we get `[ 1, 2 ]`. Cool. Using those values, calculate the x-coordinate of their intersection point:

```js
x = (b2 - b1) / (m1 - m2)
```

It should come out to `-1.7`. Again, if you find the intersection with pen and paper, you can verify that this value is correct. Now let's find the y-coordinate in each of two ways:

```js
m1 * x + b1
m2 * x + b2
```

For the first, you see that we arrive at `y=0.3`. This is the method used when we rum `s1.crosses(s2)`, but for the second we get something like `y=0.30000000000000004`. You can find a pretty decent explanation of why this is happening in this JavaScript.io article on "Imprecise Calculations": https://javascript.info/number#imprecise-calculations.

In the latter case, where the y-coordinate was 0.3 plus some, our `contains` function is going to turn up `false`, since `0.30000000000000004 <= 0.3` is `false`. It's almost never safe to use equality or inequality comparisons directly like this on floating point numbers. Instead, to check whether two floating point numbers are equal, you should allow for some approximation.

In our _utils.js_ module, let's add a few functions to help with these floating point number comparisons.

_utils.js_:
```js
...

// Floating point math is, literally, the devil. So, let's use a threshold
// of 6 decimal places for all of our numbers.
function eq(a, b, t=0.000001) { return (b - t <= a && a <= b + t) }
function lte(a, b, t=0.000001) { return (a <= b + t) }
function gte(a, b, t=0.000001) { return (a >= b - t) }

export { eq, lte, gte, zipLongest }
```

Using these, of two numbers are equal out to 6 decimal places, we'll count them as actually equal. Why 6 places? We have to pick some threshold. The more decimal places we require to be equal, the more likely imprecision will affect us. I just made a call that 6 places of precision was enough.

Now let's use these to improve our `Segment.contains` method. At the top of the _geometry.js_ module, add:

```js
import { eq, lte, gte } from './utils.js'
```

Then, replace the `contains` method with the following:

```js
contains(point) {
  const x1 = this.p1.x
  const y1 = this.p1.y
  const x2 = this.p2.x
  const y2 = this.p2.y

  const {x, y} = point
  const [m, b] = this.slopeintercept
  if (b === null && !eq(x, x1)) { return false }
  if (b !== null && !eq(y, m * x + b)) { return false}

  let lteChain = (a, b, c) => lte(a, b) && lte(b, c)
  let gteChain = (a, b, c) => gte(a, b) && gte(b, c)

  return (
    ( lteChain(x1, x, x2) || gteChain(x1, x, x2) ) &&
    ( lteChain(y1, y, y2) || gteChain(y1, y, y2) )
  )
}
```

### Normal Vectors

* In physics, a surface exerts a force perpendicular to itself. This direction is "normal".
* Our sim is implementing physics-lite.
* Add `normal` to `Boundary`

### Calculating a New Velocity After a Bounce

* Add a `bounce` method to `Agent`

----------

Things to add:
- Explain what floating point arithmetic will do, and add `contains`
- `Point.offset` vector magnitude check
- `Vector.unit`, `plus`, and `times` magnitude optimizations
- `Segment.normal` caching
- `Segment` `normal` constructor null setting (because of JIT compiling)


Outline:
* How do we know when an agent has collided with a boundary
* What should we do when an agent collides with a boundary