---
layout: page
title: Initial Setup
permalink: initial-setup
nav_order: 2
---

# Initial Setup

When I got started, I put little bit of time into what I wanted the layout of my project to be. I know I want it to be responsive (why not). I know I want to show a simulation, and graphs that are updating as the simulation runs. I know that I want to be able to tweak some parameters in the simulation directly from the UI.

Here's a sketch of what's in my mind:

![Initial sketch of UI](images/initial-sketch.jpg)

What we're looking at here (apologies for my sketches) are two versions of the same interface, in each version there's a large simulation area (on the left in the _Desktop_ sketch, and at the top in the _Mobile_ sketch) overlayed by a collapsible settings box. In the settings box there are sliders to adjust simulation parameters (we'll get to that later).

Opposite the simulation area (to the right, or below) is a set of charts that the user can scroll through. We will also some back to what these charts will show.

## HTML/CSS Structure

Let's create our initial simulation page.

```html
<!DOCTYPE html>

<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Flatten-The-Curve Simulation</title>
</head>

<body>

  <section id="simulation-section">
    <svg id="simulation" viewBox="0 0 1200 1200"></svg>
    <div id="simulation-controls"></div>
  </section>

  <section id="summary-section">
  </section>

</body>
</html>
```

We're going to start with the simulation section first, so we'll just leave a stub for the summary section and the simulation controls. I'm going to skip explaining all but a few of the above pieces here.

* The viewport `meta` tag:

  ```html
  <meta name="viewport" content="width=device-width, initial-scale=1">
  ```

  This makes the page [look "right" on a mobile device](https://developer.mozilla.org/en-US/docs/Mozilla/Mobile/Viewport_meta_tag).

* The SVG simulation container:

  ```html
  <svg id="simulation" viewBox="0 0 1200 1200"></svg>
  ```

  There are a few ways that we could choose to draw the dots in our simulation: using SVG, using a canvas element, or using something like WebGL. That list is probably in order of difficulty. There are [tradeoffs](http://dataquarium.io/svg-canvas-webgl/) in either case. For the sake of relative simplicity, we're going to use SVG elements. Using SVG is also going to help us with visibility into what our code is doing later. If we play our cards right, it should be pretty simple to switch out for either of the other choices down the line.
