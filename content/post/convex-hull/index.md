---
title: Convex Hull
description: What is the smallest convex shape to contain a set of points?
slug: convex-hull
date: 2023-07-19 00:00:00+0000
image: metal.jpeg
categories:
    - computational geometry
    - algorithms
tags:
    - interactive
    - graham scan
    - sorting
---

{{<raw_html>}}
<style>
html, body {
  margin: 0;
  padding: 0;
  top: 0.5px;
  bottom: 0.5px;
  width: 100%;
  height: 100%;
}

canvas {
  width: 100%;
  height: 100%;
}
</style>
<html>
  <body onload="init();" onresize="updateDimensions();">
    <canvas id="canvas"></canvas>
  </body>
</html>

<script>
const NUM_EX = 5;
function init() {
  window.canvas = document.getElementById("canvas");
  window.ex = Array.from({length: NUM_EX}, (_, i) => document.getElementById("ex" + i));
  window.ctx = window.canvas.getContext("2d");

  window.pts = Array.from({length: 10}, () => [Math.random(), Math.random()]);
      
  window.canvas.addEventListener("mousemove", onMouseMove, true);
  window.canvas.addEventListener("mouseup", onMouseUp, true);
  window.canvas.addEventListener("mousedown", onMouseDown, true);
  window.canvas.addEventListener("touchmove", onMouseMove, true);
  window.canvas.addEventListener("touchend", onMouseUp, true);
  window.canvas.addEventListener("touchstart", onMouseDown, true);
  window.canvas.addEventListener("mouseleave", onMouseLeave, true);
  window.canvas.addEventListener("mouseenter", onMouseEnter, true);

  window.mouseX = 0;
  window.mouseY = 0;
  window.mouseDown = false;
  window.mousePt = null;
  window.mouseInFrame = false;
  window.mouseLeeway = 1;
  updateConvexHull();
  updateDimensions();
}

function onMouseMove() {
  window.mouseInFrame = true;
  var mouseX, mouseY;
  if (event.targetTouches && event.changedTouches) {
    mouseX = (event.targetTouches[0] ? event.targetTouches[0].pageX : event.changedTouches[event.changedTouches.length-1].pageX) - window.canvas.getBoundingClientRect().left;
    mouseY = (event.targetTouches[0] ? event.targetTouches[0].clientY : event.changedTouches[event.changedTouches.length-1].clientY) - window.canvas.getBoundingClientRect().top;
    window.mouseLeeway = 8;
  } else {
    mouseX = window.event.clientX - window.canvas.getBoundingClientRect().left;
    mouseY = window.event.clientY - window.canvas.getBoundingClientRect().top;
    window.mouseLeeway = 1;
  }
  if (window.mousePt !== null) {
    let scaledDiff = windowUnscalePt([mouseX - window.mouseX, mouseY - window.mouseY]);
    window.pts[window.mousePt][0] += scaledDiff[0];
    window.pts[window.mousePt][1] += scaledDiff[1];
    updateConvexHull();
  }
  window.mouseX = mouseX;
  window.mouseY = mouseY;
  redraw();
}

function onMouseDown(e) {
  onMouseMove();
  e.preventDefault();
  window.mouseDown = true;
  let minDist = Infinity;
  window.pts.forEach((pt, i) => {
    let scaledPt = windowScalePt(pt);
    let dist = Math.pow(window.mouseX - scaledPt[0], 2) + Math.pow(window.mouseY - scaledPt[1], 2);
    if (dist <= window.mouseLeeway * Math.pow(window.dot_dim, 2) && dist <= minDist) {
      window.mousePt = i;
      minDist = dist;
    }
  });
  redraw();
}

function onMouseUp() {
  window.mouseDown = false;
  if (window.mousePt !== null) {
    let x = window.pts[window.mousePt][0];
    let y = window.pts[window.mousePt][1];
    window.pts[window.mousePt][0] = Math.min(Math.max(x, 0), 1);
    window.pts[window.mousePt][1] = Math.min(Math.max(y, 0), 1);
    window.mousePt = null;
  }
  redraw();
}

function onMouseLeave() {
  window.mouseInFrame = false;
  onMouseUp();
}

function onMouseEnter() {
  window.mouseInFrame = true;
}

function redraw() {
  window.ctx.clearRect(0, 0, window.canvas.width, window.canvas.height);
  window.ctx.fillStyle = 'white';
  window.ctx.strokeStyle = 'white';
  window.ctx.lineWidth = window.mouseInFrame ? 2 : 1;
  window.ctx.strokeRect(0, 0, window.canvas.width, window.canvas.height);
  window.ctx.lineWidth = 1;

  window.pts.forEach(pt => {
    window.ctx.beginPath();
    let scaledPt = windowScalePt(pt);
    window.ctx.arc(scaledPt[0], scaledPt[1], window.dot_dim, 0, 2 * Math.PI);
    window.ctx.stroke();
    if (Math.pow(window.mouseX - scaledPt[0], 2)
      + Math.pow(window.mouseY - scaledPt[1], 2)
      <= window.mouseLeeway * Math.pow(window.dot_dim, 2)) {
      window.ctx.fill();
    }
  });

  window.ctx.beginPath();
  let scaledStart = windowScalePt(window.ch[0]);
  window.ctx.moveTo(scaledStart[0], scaledStart[1]);
  for (let i = 1; i < window.ch.length; i++) {
    let scaledStart = windowScalePt(window.ch[i]);
    window.ctx.lineTo(scaledStart[0], scaledStart[1]);
  }
  window.ctx.closePath();

  window.ctx.fillStyle = "rgba(255, 0, 0, 0.2)";
  window.ctx.fill();
  for (let i = 0; i < NUM_EX; i++) {
    window["ex" + i + "redraw"]();
  }
}

function updateDimensions() {
  window.width = window.canvas.getBoundingClientRect().width;
  window.height = window.width / 3;
  window.canvas.width = window.width;
  window.canvas.height = window.height;
  window.dot_dim = window.width > 600 ? 6 : 5;
  for (let i = 0; i < NUM_EX; i++) {
    window.ex[i].width = window.width;
    window.ex[i].height = window.height;
  }
  redraw();
}

function updateConvexHull() {
  let min_elem = window.pts.reduce(
    (min, p) => p[0] < min[0] ? p
    : (p[0] == min[0] && p[1] < min[1] ? p
    : min), window.pts[0]
  );
  let pts = window.pts.filter(pt => pt !== min_elem)
  pts.sort((a, b) => {
    a_slope = (a[1] - min_elem[1]) / (a[0] - min_elem[0]);
    b_slope = (b[1] - min_elem[1]) / (b[0] - min_elem[0]);
    if (isNaN(a_slope)) {
        a_slope = Math.sign(a[1] - min_elem[1]) * Infinity;
    }
    if (isNaN(b_slope)) {
        b_slope = Math.sign(b[1] - min_elem[1]) * Infinity;
    }
    if (a_slope === b_slope) {
        return 0;
    }
    return a_slope - b_slope;
  });

  let ch = [min_elem];
  pts.forEach(pt => {
    if (ch.length < 2) {
      ch.push(pt);
      return;
    }
    while (ch.length > 1) {
      let a = [
        ch[ch.length - 1][0] - ch[ch.length - 2][0],
        ch[ch.length - 1][1] - ch[ch.length - 2][1]
      ];
      let b = [
        pt[0] - ch[ch.length - 1][0],
        pt[1] - ch[ch.length - 1][1]
      ];
      let det = a[0] * b[1] - b[0] * a[1];
      if (det < 0) {
        ch.pop();
      } else {
        break;
      }
    };
    ch.push(pt);
  });
  window.ch = ch;
}

function windowScalePt(pt) {
  return [pt[0] * window.canvas.width, pt[1] * window.canvas.height];
}

function windowUnscalePt(pt) {
  return [pt[0] / window.canvas.width, pt[1] / window.canvas.height];
}
</script>
{{</raw_html>}}

{{<box tip>}}
Drag the points!
{{</box>}}

## Introduction

> Given a set of points, what is the smallest convex shape that encloses all of them? What is the boundary of this shape?

This is the convex hull problem.

## 2D Convex Hull

### Observations

Notice that given a convex hull, all points lie on the same side of every edge.

{{<box info>}}
```goat
     o------------------->o
    /               o      ^
   /                        \
  / o         o              o
 v                           ^
o         o                  |
|                o           o
|     o                     ^
v                       o  /
o<------------------------o
```
Observe that for every edge, all points lie on the same side (right for top and bottom; left for all other).
{{</box>}}

### Idea 1: Add Edges with All Points on the same side

#### Naive Algorithm
$$
\begin{align*}
&\texttt{bool allPointsOnSameSideOf($e$):}\newline
&\texttt{\qquad for all other points $p\not\in e$:}\newline
&\texttt{\qquad \qquad if any $p$ on different side of $e$:}\newline
&\texttt{\qquad \qquad \qquad return false}\newline
&\texttt{\qquad return true}\newline
&\newline
&\texttt{for all points $p1$:}\newline
&\texttt{\qquad for all other points $p2$:}\newline
&\texttt{\qquad \qquad consider edge $e$ = ($p1$, $p2$)}\newline
&\texttt{\qquad \qquad if allPointsOnSameSideOf($e$):}\newline
&\texttt{\qquad \qquad \qquad add $e$ to convex hull}\newline
\end{align*}
$$

Checking if all points are on the same side is a linear ($\mathcal{O}(n)$) operation. Iterating through all possible edges is a quadratic operation ($\mathcal{O}(n^2)$). Thus, this algorithm has cubic time complexity ($\mathcal{O}(n^3)$).

#### Observation

We are checking all possible edges, but really edges are contiguous on a hull ($A\rightarrow B\rightarrow C\rightarrow A$ like a chain).

But what about the first edge? Here's another observation: the convex hull must contain the leftmost ($\min_x$) point!

{{<box info>}}If there are multiple leftmost points, then they must all be on the convex hull. Otherwise, that point would not be contained in the convex hull.{{</box>}}

With this chain in mind, we shouldn't be iterating over all points for the next: in a chain, the next point should be "close" to the current pivot point. Could we precompute with a sort in some way?

### Graham Scan
#### Algorithm
$$
\begin{align*}
&\texttt{let hull = [$\min_x$]}\newline
&\texttt{sort all points by $\theta$ with $\min_x$}\newline
&\newline
&\texttt{for all points $p\neq\min_x$:}\newline
&\texttt{\qquad add $p$ to hull}\newline
&\texttt{\qquad while hull (with $p$) is no longer convex:}\newline
&\texttt{\qquad \qquad pop hull}\newline
\end{align*}
$$
Convex meaning that the hull only "leans" in one direction (always curving left OR always curving right). This can easily be checked by verifying that the current point $p$ is on the same side of the previous edge (or vacuously true if there is no previous edge).

#### Example

{{<raw_html>}}
</style>
<html>
  <body onload="init();" onresize="updateDimensions();">
    <canvas id="ex0"></canvas>
  </body>
</html>

<script>
function ex0redraw() {
  let canvas = window.ex[0];
  let ctx = canvas.getContext("2d");
  ctx.fillStyle = 'white';
  ctx.strokeStyle = 'white';
  ctx.clearRect(0, 0, window.canvas.width, window.canvas.height);
  ctx.strokeRect(0, 0, window.canvas.width, window.canvas.height);

  let pts = [
    [0.09349743488591489, 0.586267349659145],
    [0.40060336938038543, 0.8863856711116301],
    [0.7331262185197241, 0.851984423840718],
    [0.41631943954503126, 0.5888341099600415],
    [0.31631943954503126, 0.5288341099600415],
    [0.5151129707167391, 0.2959397175037858],
    [0.4494122161398628, 0.09309081188615609],
    [0.13235547336082165, 0.3696404944957633]
  ];

  ctx.font = (4 * window.dot_dim) + "px bold serif";
  pts.forEach((pt, i) => {
    ctx.beginPath();
    let scaledPt = windowScalePt(pt);
    ctx.arc(scaledPt[0], scaledPt[1], window.dot_dim, 0, 2 * Math.PI);
    ctx.fillStyle = 'black';
    if (i === 0) {
      ctx.fillStyle = 'red';
    }
    ctx.fill();
    ctx.fillStyle = 'white';
    if (i !== 0) {
      ctx.fillText(i, scaledPt[0], scaledPt[1])
    }
  });

  let hull = [
    pts[0]
  ];

  ctx.beginPath();
  let scaledStart = windowScalePt(hull[0]);
  ctx.moveTo(scaledStart[0], scaledStart[1]);
  for (let i = 1; i < hull.length; i++) {
    let scaledStart = windowScalePt(hull[i]);
    ctx.lineTo(scaledStart[0], scaledStart[1]);
  }
  ctx.closePath();
  ctx.stroke();
}
</script>
{{</raw_html>}}

Here, the leftmost point is highlighted in red. We sort the other points by their angle with this point (ordered numbering).

{{<raw_html>}}
</style>
<html>
  <body onload="init();" onresize="updateDimensions();">
    <canvas id="ex1"></canvas>
  </body>
</html>

<script>
function ex1redraw() {
  let canvas = window.ex[1];
  let ctx = canvas.getContext("2d");
  ctx.fillStyle = 'white';
  ctx.strokeStyle = 'white';
  ctx.clearRect(0, 0, window.canvas.width, window.canvas.height);
  ctx.strokeRect(0, 0, window.canvas.width, window.canvas.height);

  let pts = [
    [0.09349743488591489, 0.586267349659145],
    [0.40060336938038543, 0.8863856711116301],
    [0.7331262185197241, 0.851984423840718],
    [0.41631943954503126, 0.5888341099600415],
    [0.31631943954503126, 0.5288341099600415],
    [0.5151129707167391, 0.2959397175037858],
    [0.4494122161398628, 0.09309081188615609],
    [0.13235547336082165, 0.3696404944957633]
  ];

  let hull = [
    pts[0],
    pts[1],
    pts[2],
    pts[3],
    pts[4]
  ];

  ctx.beginPath();
  let scaledStart = windowScalePt(hull[0]);
  ctx.moveTo(scaledStart[0], scaledStart[1]);
  for (let i = 1; i < hull.length; i++) {
    let scaledStart = windowScalePt(hull[i]);
    ctx.lineTo(scaledStart[0], scaledStart[1]);
  }
  ctx.stroke();

  pts.forEach((pt, i) => {
    ctx.beginPath();
    let scaledPt = windowScalePt(pt);
    ctx.arc(scaledPt[0], scaledPt[1], window.dot_dim, 0, 2 * Math.PI);
    ctx.fillStyle = 'black';
    if (i === 4) {
      ctx.fillStyle = 'red';
    }
    ctx.fill();
    ctx.fillStyle = 'white';
  });
}
</script>
{{</raw_html>}}

After a few iterations, our hull is still convex (always left leaning).

{{<raw_html>}}
</style>
<html>
  <body onload="init();" onresize="updateDimensions();">
    <canvas id="ex2"></canvas>
  </body>
</html>

<script>
function ex2redraw() {
  let canvas = window.ex[2];
  let ctx = canvas.getContext("2d");
  ctx.fillStyle = 'white';
  ctx.strokeStyle = 'white';
  ctx.clearRect(0, 0, window.canvas.width, window.canvas.height);
  ctx.strokeRect(0, 0, window.canvas.width, window.canvas.height);


  let pts = [
    [0.09349743488591489, 0.586267349659145],
    [0.40060336938038543, 0.8863856711116301],
    [0.7331262185197241, 0.851984423840718],
    [0.41631943954503126, 0.5888341099600415],
    [0.31631943954503126, 0.5288341099600415],
    [0.5151129707167391, 0.2959397175037858],
    [0.4494122161398628, 0.09309081188615609],
    [0.13235547336082165, 0.3696404944957633]
  ];

  let hull = [
    pts[0],
    pts[1],
    pts[2],
    pts[3],
    pts[4],
    pts[5]
  ];

  ctx.beginPath();
  let scaledStart = windowScalePt(hull[0]);
  ctx.moveTo(scaledStart[0], scaledStart[1]);
  for (let i = 1; i < hull.length; i++) {
    let scaledStart = windowScalePt(hull[i]);
    ctx.lineTo(scaledStart[0], scaledStart[1]);
  }
  ctx.stroke();

  pts.forEach((pt, i) => {
    ctx.beginPath();
    let scaledPt = windowScalePt(pt);
    ctx.arc(scaledPt[0], scaledPt[1], window.dot_dim, 0, 2 * Math.PI);
    ctx.fillStyle = 'black';
    if (i === 5) {
      ctx.fillStyle = 'red';
    }
    ctx.fill();
    ctx.fillStyle = 'white';
  });
}
</script>
{{</raw_html>}}

Now, the hull's convexitivity breaks.

{{<raw_html>}}
</style>
<html>
  <body onload="init();" onresize="updateDimensions();">
    <canvas id="ex3"></canvas>
  </body>
</html>

<script>
function ex3redraw() {
  let canvas = window.ex[3];
  let ctx = canvas.getContext("2d");
  ctx.fillStyle = 'white';
  ctx.strokeStyle = 'white';
  ctx.clearRect(0, 0, window.canvas.width, window.canvas.height);
  ctx.strokeRect(0, 0, window.canvas.width, window.canvas.height);


  let pts = [
    [0.09349743488591489, 0.586267349659145],
    [0.40060336938038543, 0.8863856711116301],
    [0.7331262185197241, 0.851984423840718],
    [0.41631943954503126, 0.5888341099600415],
    [0.31631943954503126, 0.5288341099600415],
    [0.5151129707167391, 0.2959397175037858],
    [0.4494122161398628, 0.09309081188615609],
    [0.13235547336082165, 0.3696404944957633]
  ];

  let hull = [
    pts[0],
    pts[1],
    pts[2],
    pts[3],
    pts[5]
  ];

  ctx.beginPath();
  let scaledStart = windowScalePt(hull[0]);
  ctx.moveTo(scaledStart[0], scaledStart[1]);
  for (let i = 1; i < hull.length; i++) {
    let scaledStart = windowScalePt(hull[i]);
    ctx.lineTo(scaledStart[0], scaledStart[1]);
  }
  ctx.stroke();

  pts.forEach((pt, i) => {
    ctx.beginPath();
    let scaledPt = windowScalePt(pt);
    ctx.arc(scaledPt[0], scaledPt[1], window.dot_dim, 0, 2 * Math.PI);
    ctx.fillStyle = 'black';
    if (i === 5) {
      ctx.fillStyle = 'red';
    }
    ctx.fill();
    ctx.fillStyle = 'white';
  });
}
</script>
{{</raw_html>}}

Still broken (we only have to check that the current red point is on the left side of the 2nd-to-last edge).

{{<raw_html>}}
</style>
<html>
  <body onload="init();" onresize="updateDimensions();">
    <canvas id="ex4"></canvas>
  </body>
</html>

<script>
function ex4redraw() {
  let canvas = window.ex[4];
  let ctx = canvas.getContext("2d");
  ctx.fillStyle = 'white';
  ctx.strokeStyle = 'white';
  ctx.clearRect(0, 0, window.canvas.width, window.canvas.height);
  ctx.strokeRect(0, 0, window.canvas.width, window.canvas.height);


  let pts = [
    [0.09349743488591489, 0.586267349659145],
    [0.40060336938038543, 0.8863856711116301],
    [0.7331262185197241, 0.851984423840718],
    [0.41631943954503126, 0.5888341099600415],
    [0.31631943954503126, 0.5288341099600415],
    [0.5151129707167391, 0.2959397175037858],
    [0.4494122161398628, 0.09309081188615609],
    [0.13235547336082165, 0.3696404944957633]
  ];

  let hull = [
    pts[0],
    pts[1],
    pts[2],
    pts[5]
  ];

  ctx.beginPath();
  let scaledStart = windowScalePt(hull[0]);
  ctx.moveTo(scaledStart[0], scaledStart[1]);
  for (let i = 1; i < hull.length; i++) {
    let scaledStart = windowScalePt(hull[i]);
    ctx.lineTo(scaledStart[0], scaledStart[1]);
  }
  ctx.stroke();

  pts.forEach((pt, i) => {
    ctx.beginPath();
    let scaledPt = windowScalePt(pt);
    ctx.arc(scaledPt[0], scaledPt[1], window.dot_dim, 0, 2 * Math.PI);
    ctx.fillStyle = 'black';
    if (i === 5) {
      ctx.fillStyle = 'red';
    }
    ctx.fill();
    ctx.fillStyle = 'white';
  });
}
</script>
{{</raw_html>}}

Convex!

---

If we continue the algorithm, we will eventually reach the full convex hull.

### Analysis

Determining the leftmost point just requires a linear scan ($\mathcal{O}(n)$). Since determining the angle between two points takes constant time ($\mathcal{O}(1)$), the initial sort has the normal time complexity $\mathcal{O}(n \log n)$.

Every point will be the current point (red dot in example) exactly once, so this iteration has linear time complexity ($\mathcal{O}(n)$).

What about popping non-convex points? Observe that any point can be popped at most once! So it has linear time complexity ($\mathcal{O}(n)$).

$$\mathcal{O}(n) + \mathcal{O}(n \log n) + \mathcal{O}(n) + \mathcal{O}(n)$$ $$\subseteq\mathcal{O}(n \log n)$$

{{<box important>}}A bit of notation abuse, but I think the point still gets across.{{</box>}}

## Above and Beyond

{{<box important>}}
Anything in this section serves as very light introductions to convex hull extensions. You should treat them as just introductions and feel free to explore further beyond this blog post.
{{</box>}}

There are various algorithms for solving this problem in $d$-dimensions. For the 3D problem, there exists a deterministic divide-and-conquer $\mathcal{O}(n \log n)$ algorithm (but is notoriously difficult, especially the merge step).

### Chan's Algorithm

[Chan's algorithm](https://en.wikipedia.org/wiki/Chan%27s_algorithm) is an output-sensitive algorithm for calculating the convex hull in 2D and 3D space.

It relies on existing $\mathcal{O}(n \log n)$ algorithms in those spaces (Graham Scan, divide-and-conquer, etc.) and uses them to solve subpartitions of the original point set. It then combines these mini-convex hulls to solve the full problem.

It has time complexity $\mathcal{O}(n \log h)$, where $h$ is the number of vertices in the solution convex hull.

### Quick Hull

For the general $d$, [Quick Hull](https://en.wikipedia.org/wiki/Quickhull) is a [Las Vegas algorithm](https://en.wikipedia.org/wiki/Las_Vegas_algorithm) that has a time complexity of $\mathcal{O}(n \log n)$ in expectation for the 2D and 3D case.

{{<box info>}}The $d$-dimensional generalization adds a time complexity factor of $n^{\lfloor\frac{d}{2}\rfloor}$. I don't have great intuition on this expression, but it has to do with the number of facets on a $d$-dimensional object (increasing recursion breadth).{{</box>}}

I don't have great exposure to Quick Hull, but the general idea is that we begin with a "starter hull" (subset of the convex hull; encompasses a large percentage of points in expectation).

{{<box info>}}
For the 2D case, this hull might have $p_1=\min_x$, $p_2=\max_x$, and $p_3=\max_\text{dist}(p_1p_2)$ (the point that is furthest from the line $p_1p_2$).

For the 3D case, this hull might have the same points as in the 2D case plus $p_4=\max_\text{dist}(p_1p_2p_3)$ (the point that is furthest from the plane $p_1p_2p_3$)
{{</box>}}

Then, for each facet on the current hull (line in 2D; plane in 3D), add the point $p$ (on the other side of the hull) farthest from the facet.

This inclusion may break the convexitivity of the current hull. Perform [BFS](https://en.wikipedia.org/wiki/Breadth-first_search) to determine the "horizon" of $p$ and remove all visited points not in the horizon.

{{<box info>}}
The horizon is what you might be able to see if you were to put your eye at $p$. This can be checked using linear algebra magic.

If you can see a point $a$ behind a point $b$, then that means $b$ is not in the horizon and causes the hull to be concave.
{{</box>}}

Recursively repeat the previous two steps until every point is enclosed in the hull.
