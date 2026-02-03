---
layout: post
title: Closest String Problem
subtitle: Final project of my Computer Science Graduation
gh-repo: ThiagoFBastos/Closest-String-Problem
gh-badge: [star, fork, follow]
tags: [test]
comments: true
mathjax: true
author: Thiago Felipe Bastos da Silva
---

## Purpose
The Closest String Problem has many applications in bioinformatics and your target is to find a string t, given a set S with strings of same length, such that maximum Hamming Distance between t and all $S_i$, $1 \le i \le |S|$, be minimum. Since it belongs the np-hard class is no known a polynomial algorithm in order to solve it. Therefore, to get a solution uses some exact method with restriction or uses some heuristic that not guarantees optimality but the answer is obtained in a reasonable time. Therefore, this work proposes to create algorithms based on metaheuristics already known as: Simulated Annealing, Iterated Local Search and Genetic Algorithm and compare the results with what has already been obtained in the literature.

### Definition of the problem
Let $S$ be a sequence of strings ${ S_1, S_2, S_3, \ldots, S_n }$ with $|S_i| = m$, whose characters belong to an alphabet $\Sigma$. The Closest String Problem consists of finding a string $T$, also of length $m$, such that the maximum Hamming distance between $T$ and all strings in the input set $S$ is minimized, i.e.,  $T$ = $argmin_T$ ${max_{1 \le i \le n}{d(S_i, T)}}$.

### Heuristics of initial solution

There are four heuristics for initial solution that are used by the metaheuristics that i have implemented. Here are them:
1. The algorithm 1 consists of generating a string in which each position $1 \le j \le m$ contains a character randomly chosen from among the characters present in column $j$ of the input strings.
2. Algorithm 2, which was based on the work by (SANTOS, 2018) in his master’s thesis, follows a greedy strategy for constructing the string and, for this purpose, randomly selects, for each position, one of the most frequent characters at that position.
3. Algorithm 3 consists of generating a string $T$ by assigning each of its positions to one of the input strings in such a way that the number of positions assigned to any string differs by at most one unit from the others. Once the assignment is performed, the characters at the positions assigned to the input strings are copied to the corresponding positions in the string $T$.
4. Algorithm 4 consists of generating a string $T$ by merging the input strings in a predefined order. Each merge assigns $\frac{1}{i}$ of the positions that have different characters to one of the strings, and the remaining positions to the other.

### Metaheuristics

- Simulated Annealing
- Parallel Simulated Annealing
- ILS (Iterated Local Search) using as local search the simulated annealing
- Generic Algorithm

### Project

The project can be found at [link](https://github.com/ThiagoFBastos/Closest-String-Problem)

### Technologies

- C++17
- GNU Make
- Pthread
- Vectorized instructions
