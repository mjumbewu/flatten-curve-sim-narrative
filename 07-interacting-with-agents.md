---
layout: page
title: Interacting With Other Agents
permalink: interacting-with-agents
nav_order: 7
---

# Interacting With Other Agents

Bouncing off of other agents is going to be very similar to bouncing off of boundaries. The main difference is that the collision **normal** between two agents is going to be dependent on how the agents collide.

A ball has an essentially infinite number of normal vectors, each starting at the center of the ball and continuing in every direction around that center. Each of these normals is perpendicular to the surface of the ball.

{% include illustrations/ball_infinite_normals.html %}

When two balls collide, the centers of the balls are the endpoints of a line segment that contains the point of collision between the two balls.

{% include illustrations/balls_collision_point.html %}

If we're trying to figure out what happens to the ball `a` after it collides with ball `b`, and we can calculate the normal vector to the surface of `b` at the point of collision, then we can treat this the same (essentially) as we would a collision with a boundary.

To calculate a vector from two points, let's add a method called `diff` to the `Point` class. This method will be a kind of inverse to the `offset` method, in that, for a point `p` and a vector `v`:

```js
p.offset(v).diff(p) == v
```

In other words, `p.offset(v)` will produce a new point (let's call it `p2`) that is shifted from `p` according to the direction and magnitude of `v`. Then, if we run `p2.diff(p)` we get a new vector whose magnitude and direction is the same as the original vector `v`. The `diff` method is simply implemented as:

```js
diff(other) {
  return new Vector(
    this.x - other.x,
    this.y - other.y,
  )
}
```

We can now use this to create a method that will check for collisions between a given agent and all the other agents in a world. Add the following to the _model.world.js_ module:

```js
function bounceAgentOffOtherAgents(agent, otherAgents) {
  function areAgentsColliding(other) {
    if (other === agent) { return false }

    const minCollisionDist = agent.radius + other.radius
    const diff = agent.position.diff(other.position)
    return (
      agent.velocity.dot(diff) < 0 &&
      diff.magnitude <= minCollisionDist
    )
  }

  function getAgentCollisionNormal(other) {
    return agent.position.diff(other.position).unit
  }

  const collisionNormals = otherAgents
    .filter(areAgentsColliding)
    .map(getAgentCollisionNormal)

  return agent.bounce(collisionNormals)
}
```

Finally, amend the `World.step` method to call this `bounceAgentOffOtherAgents` function in addition to `bounceAgentOffBoundaries`:

```js
step(Δt=1) {
  const agents = this.agents
    .map((a, i, agents) => bounceAgentOffOtherAgents(a, agents))
    .map(a => bounceAgentOffBoundaries(a, a.step(Δt), this.boundaries))
  const time = this.time + Δt

  return new World({
    ...this,
    agents,
    time,
  })
}
```

Now, having gotten this far, our agents should be bouncing off the walls and off of each other.

----------

