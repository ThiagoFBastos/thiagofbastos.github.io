---
layout: post
title: Isotree
subtitle: An algorithm for finding all non-isomorphic trees of a given size.
tags: [algorithms, backtracking, graphs, parallel algorithms]
author: Thiago Felipe Bastos da Silva
---

After using the Prüfer code in my Scientific Initiation project to find all non-isomorphic trees of a given size, I came up with an idea to improve this algorithm, since it was slow and I could not generate the trees-or even count them for larger values of n. Therefore, I proposed a backtracking algorithm with pruning to reduce the computation time.

<br><br>

This backtracking algorithm is a constructive approach that builds the tree level by level while enforcing specific rules to reduce the search space. Moreover, to avoid generating duplicate trees, we used a hash table from the GNU C++ library, gp_hash_table, which is well known in the competitive programming community for being faster than std::unordered_set.

<br><br>

One might question whether the algorithm could consume excessive memory, given that the number of trees grows rapidly and the method requires $O(n)$ space. However, this issue is mitigated by the fact that the trees considered are relatively small (since the number of non-isomorphic trees increases very quickly with n), allowing us to encode and store the trees efficiently as integers.

### Pruning rules used by the algorithm
1. The tree is rooted.
2. Vertices closer to the root have smaller labels than vertices farther from it.
3. For two vertices at the same level, the vertex with the larger label must have all its children labeled with values greater than those of the other vertex.
4. For two sibling vertices, the one with the larger label must have greater height.
5. The root belongs to the center of the tree; therefore, the height difference between the two tallest subtrees is at most one.


### Parallel version

A parallel version of this algorithm was implemented using OpenMP. This approach is straightforward, since each thread independently runs the serial algorithm on a distinct subtree of the root.

### Achievements

Using both the serial and parallel versions of the algorithm, we can enumerate all trees with $n \le 22$ in a reasonable amount of time. This corresponds to 5,623,756 trees. Moreover, the algorithm is capable of handling trees with up to 24 vertices, although this was not experimentally verified due to time constraints.

### Project

Tne project can be found in [link](https://github.com/ThiagoFBastos/isotree)
