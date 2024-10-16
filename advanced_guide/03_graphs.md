# Spatial components in Sampy

In Sampy, discrete spaces are encoded using directed graphs. That are, mathematical structures composed of a set of vertices linked by a set of arrows. Graphically, it looks like this.

<p align="middle">
  <img src="./assets/example_graph.png" width="25%" />
</p>

Even if the graphs used in Sampy are technically directed, in most cases if `x` and `y` are two vertices such that there is an arrow from `x` to `y`, then there is an opposite arrow from `y` to `x`. In such cases, we represent both arrows with a single straight edge, as shown in the following figure.

<p align="middle">
  <img src="./assets/two_square_grids.png" width="50%" />
</p>

## What graphs are meant to represent in Sampy



## Connections array

In mathematics, graphs are often encoded using an adjacency matrix, which are defined as follows. If `G` is a graph whose vertices are indexed with integer from 0 to `n-1` (where `n` is the number of vertices in `G`), then the adjacency matrix of `G` is the matrix `M` of size `n x n` such that `M[i, j] = 1` if there is an arrow from the vertex `i` to the vertex `j`, and 0 otherwise. 

<p align="middle">
  <img src="./assets/adjacency_matrix.png" width="50%" />
</p>