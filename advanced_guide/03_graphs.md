# Spatial components in Sampy

In Sampy, discrete spaces are encoded using directed graphs. That are, mathematical structures composed of a set of vertices linked by a set of arrows. Graphically, it looks like this.

<p align="middle">
  <img src="./assets/example_graph.png" width="25%" />
</p>

Even if the graphs used in Sampy are technically directed, in most cases if `x` and `y` are two vertices such that there is an arrow from `x` to `y`, then there is an opposite arrow from `y` to `x`. In such cases, we represent both arrows with a single straight edge, as shown in the following figure.

<p align="middle">
  <img src="./assets/two_square_grids.png" width="50%" />
</p>