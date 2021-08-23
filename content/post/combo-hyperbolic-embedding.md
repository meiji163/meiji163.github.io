---
title: "Combinatorial Hyperbolic Embeddings"
date: 2021-08-22
tags: ["machine learning", "hyperbolic"]
categories: ["machine learning", "geometry", "hyperbolic"]
katex: true
draft: true
comment: true
---

## Introduction
The [manifold hypothesis](http://colah.github.io/posts/2014-03-NN-Manifolds-Topology/#fnref1) says that most real-world datasets lie approximately on a low-dimensional manifold, 
but by some Kantian twist of fate we rarely have direct access to this manifold[^gunnar]. As a result, most machine learning techniques utilize only _local_ structure (e.g. the "loss + SGD" machine).

[^gunnar]: In some low-dimensional cases it is tractable. A notable experiment by Carlsson showed that 3x3 pixel patches on a dataset of black and white photos lay on a [Klein bottle](https://en.wikipedia.org/wiki/Klein_bottle) in \\( \mathbb{R}^9 \\)

Topological and Geometric Data Analysis (TGDA) is a burgeoning field that studies the _global_ structure of data. 
This is exciting because it opens possibilities for porting a lot of powerful (and beautiful) math to the ML realm.  
So far, TGDA has seen success in domains like... 

In this post I'll discuss hyperbolic deep learning, an active research area in TGDA (it fits under the "_G_" rather than the "_T_").   
Then ... 

## Hyperbolic Space
{{< figure src="https://uploads5.wikiart.org/images/m-c-escher/circle-limit-iv.jpg" caption="M. C. Escher, Circle Limit IV">}} 

I find it difficult to explain hyperbolic space to the geometrically uninitiated (it usually involves me going on about Riemannian metrics and curvature tensors while gesturing wildly at Escher's _Circle Limit_).
The geometry of a sphere or plane is easy to grasp, but negative curvature is unintuitive to our Euclidean brains.

Luckily, I can explain it simply if you know what a [tree](https://en.wikipedia.org/wiki/Tree_(graph_theory)) is: hyperbolic space is a _continuous version of a tree_.


### \\(\delta\\)-Hyperbolicity
Suppse \\( X \\) is a set of points and \\(d(\cdot, \cdot)\\) is a [metric](https://en.wikipedia.org/wiki/Metric_space) on \\(X\\) (a symmetric distance function that satisfies the triangle inequality). 

Define the "Gromov product" as 

$$ (x | y)_z = \frac{1}{2}(d(x,z)+d(y,z)-d(x,y)) - \delta$$

This roughly measures the distance of \\(z\\) from the geodesic (the shortest path) connecting \\(x\\) and \\(y\\).

**Definition**:  A metric space \\( (X, d)\\) is \\(\delta\\)-hyperbolic ( \\(\delta \ge 0\\) ) if 

$$ (x | y)_w \ge \min( (x|z)_w, (y|z)_w)) - \delta \quad \text{for all } x,y,z,w \in X.$$

This condition essentially says that no point in the triangle formed by \\(x,y,z\\) can be too far from one of the edges. 
In fact, it's equivalent to require all triangles in \\(X\\) to be "slim"[^geodesic]. \\( \delta \\) is the "slimness" parameter.

[^geodesic]: We assume

{{< figure src="/images/deltah.png" caption="Illustration of the delta-hyperbolic condition in the Euclidean plane. The three cirlces are incircles of the triangles xyw, xwz, and zyw. The length of the 3 red segments equal to the 3 Gromov products" >}} 

What kind of space has slim triangles (small \\(\delta\\))? A _tree_!   
For example, take an unweighted tree with its shortest-path distance. 

{{< figure src="https://graphstream-project.org/media/img/generator_overview_banana_tree.png#center" >}}

As a metric space it is 0-hyperbolic, i.e. \\( (x|y)_w \ge \min( (y|z)_w, (z,x)_w ) \\) (Proof: Exercise).
A tree has no cycles, so every triangle is just a path -- a degenerate triangle. Trees are therefore the _most_ hyperbolic of spaces; we call 0-hyperbolic metrics _tree metrics_ (note however not all tree metrics come from a _tree graph_).

Now it's easy to imagine graphs with a small \\(\delta\\): they're the "tree-like" ones with few long cycles. 

### Poincaré Disk Model
Graphs are well and good for you discrete-ophiles, but we want continuous hyperbolic spaces with derivatives and stuff! 

There are many models of hyperbolic space[^hyp], but one of the easiest to visualize is the Poincaré model. 
It is the set of points in the Euclidean disk \\( \mathbb{H}_2 = \\{ x\in \mathbb{R}^2 : |x| < 1 \\} \\)  with the metric given by

[^hyp]: More specifically, 

$$ d_H(x,y) = \cosh^{-1} \left(1 + 2\frac{|x-y|^2}{(1-|x|^2)(1-|y|^2)}\right).$$

Nevermind where this crazy function comes from, just note that the distance gets huge near the boundary circle \\( |x| \to 1 \\) ( \\( \cosh^{-1}(r) \sim \log(r) \\) grows logarithmically ).    
Geodesics (shortest paths) in this model are arcs of circles that intersect the boundary circle at right angles. As a result, triangles look like this:

{{< figure src="https://pointatinfinityblog.files.wordpress.com/2018/02/triangle5.png?w=480&h=480#center" >}}

To our Euclidean eyes, the triangles get smaller and more distorted towards the boundary, but all of these triangles are acutally the same size when measured with the Poincaré metric.

As you might suspect, triangles in the Poincaré disk are "slim," and it turns out this metric space is \\(\delta\\)-hyperbolic with \\(\delta = \log(1+\sqrt{2}) \approx 0.881 \\).
This model can be generalized easily to higher dimensions, and lower or higher \\(\delta\\).


## Hyperbolic Data

Now that we've established a correspondence "hyperbolic" ⟷   "treelike," we can ask what type of data has a hyperbolic structure.

The answer is: anything with a _hierarchical structure_. Words, social networks, hyperlink graphs, knowledge graphs, genomic data, images, even financial data -- and that's only the tip of the iceberg (see the [bibliography](#references) and references therein).   

One way to understand the hyperbolic structure of these data is in terms of categorization (or [hierarchical clustering](https://en.wikipedia.org/wiki/Hierarchical_clustering) for you data science people). Often there is a hierarchy of categories that gives rise to an underlying tree (Henri is a human; a human is a primate). However, we may get something that is only approximately a tree due to heterogenous or overlapping categories (Henri is a human; Henri is a mathematician; a mathematician is a human). 

### The Embedding Problem 

The problem is now an algorithmic one: given data, compute a hyperbolic representation. A large portion of hyperbolic learning research focuses on this problem. 

For data with a natural metric, we can formulate it as follows:
_given data \\(X \\) with a metric \\( d \\) find an embedding \\(f: (X,d) \to (\mathbb{H}, d_H) \\)_ where \\( (\mathbb{H}, d_H ) \\) is a model of hyperbolic space.
The embedding should be "good" by some measure; a natural choice is to minimize _average distortion_, defined by 

$$D_{avg} = \frac{1}{\binom{n}{2}} \sum_{x,y\in X} \frac{|d_H(f(x),f(y)) - d(x,y)|}{d(x,y)}$$

where the sum is over distinct pairs \\( \\{x,y\\} \\) and the \\(n\\) is the number of points in \\(X\\).

There are two main strategies:

1. (**SGD**) Use a loss function and gradient descent to learn the embedding
2. (**Combinatorial**) Embed the data into a (weighted) tree, then embed the tree in hyperbolic space

The drawback of the SGD stategy is the computational cost and numerical instability of doing gradient descent with the Riemannian (hyperbolic) gradient.
For the remainder of this post I'll primarily talk about the combinatorial strategy. 

### Get in the Tree!
The second part of combinatorial embedding (embed a tree in hyperbolic space) is easy. An algorithm by Sarkar [[2]](#references) embeds a tree with \\(n\\) nodes in the Poincare disk in \\( O(n) \\) time. It also guarantees that the distortion is at most \\( 1+\epsilon \\), if the edge weights are scaled appropriately.

The idea is simple: place a node at the origin and place all its children equally spaced in a circle around it. Then recursively move the children to the origin (by a hyperbolic [isometry](https://en.wikipedia.org/wiki/Hyperbolic_motion)) and repeat.

Here's an example of an embedding of a complete binary tree with 511 nodes that achieves an average distortion of 0.153:

{{< figure src="/images/hyp-bin-tree.png" >}}

The authors of [[1]](#references) provided a generalization of Sarkar's algorithm to higher dimensions. The only downside of this algorithm is it requires high precision float arithmetic (as the points tend to the boundary).

Now we can address the first part of the combinatorial embedding (embed the data into a tree), which is the more interesting problem. 

## References
[[1]] de sa et al
[[2]] sarkar
