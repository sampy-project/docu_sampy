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

In mathematics, graphs are often encoded using an adjacency matrix, which are defined as follows. If `G` is a graph whose vertices are indexed with integer from 0 to `n-1` (where `n` is the number of vertices in `G`), then the adjacency matrix of `G` is the matrix `M` of size `n x n` such that `M[i, j] = 1` if there is an arrow from the vertex `i` to the vertex `j`, and 0 otherwise. We show on the following figure an example of such a matrix.

<p align="middle">
  <img src="./assets/adjacency_matrix.png" width="50%" />
</p>

Even if adjacency matrices are extremely useful theoretical tools in mathematics and informatics, they often are memory inneficient to store graphs' structure. Indeed, in most practical cases those matrices are filled with zeros. Generally, adjacency matrices are encoded using "sparce matrices", but in our case we can do something more straightforward. Given the previous section, graphs in Sampy have the following properties.

1. A graph in sampy has a small maximal out-degree (i.e. the maximal number of arrows coming out of any vertex). Typically, this number is below 10.
2. The vast majority of the graph's vertices actually have this number as out-degree (the exceptions mostly happen at the borders of the map).

If we denote by `D` the maximal out-degree of the graph, and `n` its number of vertices, then we can encode the graph' structure in a `n x D` array `C` as follows.

1. `C` is an array of integers.
2. `C[i, :]` is essentially the list of all the vertices `v` such that there is an arrow from `i` to `v`.
3. de 

