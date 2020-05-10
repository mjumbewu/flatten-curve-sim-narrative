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

Let's create a new module for these concepts:

_geometry.js_:
```js
const π = Math.PI

// Vector
// ======
// We will define a vector in one of two ways: using its x and y components,
// or using its magnitude and direction. For the majority of our operations
// we're going to use the x and y components, so those are what we will set on
// on the object, but we'll add getters to quickly compute the magnitude and
// direction from the x and y components if needed. We will also add a getter
// for the unit of a vector (the vector with a magnitude of 1 in the same
// direction), and a few function for doing vector arithmetic (addition,
// multiplication, and "dot product").
class Vector {
  constructor(Δx, Δy) {
    this.Δx = Δx
    this.Δy = Δy
  }

  static fromPolar(magnitude, direction) {
    return new Vector(
      Math.cos(direction) * magnitude,
      Math.sin(direction) * magnitude,
    )
  }

  get iszero() {
    return this.Δx === 0 && this.Δy === 0
  }

  get magnitude() {
    return Math.sqrt(this.Δx * this.Δx + this.Δy * this.Δy)
  }

  get direction() {
    if (this.Δx === 0) {
      if (this.Δy > 0) { this._direction = π/2 }
      else         { this._direction = 3*π/2 }
    }

    let _direction = Math.atan(this.Δy / this.Δx)

    // arctan has a range between -90° and 90° (always pointing to the right).
    // When this.Δx is negative, we need to rotate the direction by 180°.
    if (this.Δx < 0) { this._direction += π }

    // Normalize the direction.
    while (this._direction < 0)    { this._direction += 2*π }
    while (this._direction >= 2*π) { this._direction -= 2*π }

    return _direction
  }

  get unit() {
    const magnitude = this.magnitude
    return new Vector(
      this.Δx / magnitude,
      this.Δy / magnitude,
    )
  }

  plus(other) {
    return new Vector(
      this.Δx + other.Δx,
      this.Δy + other.Δy,
    )
  }

  times(factor) {
    return new Vector(
      this.Δx * factor,
      this.Δy * factor,
    )
  }

  dot(other) {
    return this.Δx * other.Δx + this.Δy * other.Δy
  }
}

// Point
// =====
// A point is simple: it's just an x and y coordinate. We also want to be able
// to quickly offset a point by some vector amount.
class Point {
  constructor(x, y) {
    this.x = x
    this.y = y
  }

  offset(vector) {
    return new Point(
      this.x + vector.Δx,
      this.y + vector.Δy,
    )
  }
}

// Segment
// =======
// A segment is also relatively simple: just goes from one x/y coordinate to
// another. We will add a couple of getters here as well: one for the slope of
// the segment, as this will come in handy down the line, and another for the
// "normal" vector. A normal vector of a segment is the unit vector (i.e.,
// vector of length 1) that is orthogonal (i.e., perpendicular) to the segment.
class Segment {
  constructor(x1, y1, x2, y2) {
    this.x1 = x1
    this.y1 = y1
    this.x2 = x2
    this.y2 = y2
  }

  get normal() {
    const orthogonal = new Vector(
      this.y2 - this.y1,
      this.x1 - this.x2,
    )
    return orthogonal.unit
  }

  get slope() {
    return 1.0 * (this.y2 - this.y1) / (this.x2 - this.x1)
  }
}

export { Point, Segment, Vector }
```

Things to add:
- `Point.offset` vector magnitude check
- `magnitude` and `direction` caching
- `Vector` constructor parameters `magnitude` and `direction`
- `Vector.unit`, `plus`, and `times` magnitude optimizations
- `Segment` `normal` constructor parameter
- `Segment.normal` caching


Outline:
* How do we know when an agent has collided with a boundary
* What should we do when an agent collides with a boundary