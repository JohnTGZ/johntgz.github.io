---
layout: post
title: "The Travelling Salesman - Dynamic Programming and Branch-and-bound method"
---

When it comes to TSP, I see rather unsatisfactory explainations out there, or rather, no easy ones for the lazy mind. Today I shall attempt to illumination this classic problem with visual representations and diagrams. There are [multiple approaches to TSP](https://en.wikipedia.org/wiki/Travelling_salesman_problem), with the most commonly discussed being [Dynamic Programming](https://www.youtube.com/watch?v=aPQY__2H3tE&ab_channel=Reducible) and [Branch and Bound](https://en.wikipedia.org/wiki/Branch_and_bound), which will be the focus today.

# Table of Contents
- [Table of Contents](#table-of-contents)
- [Brute Force?](#brute-force)
- [Dynamic Programming!](#dynamic-programming)
- [Branch and Bound](#branch-and-bound)

# Brute Force?

Time complexity: $O(n!)$

Space complexity: $O(n^2)$


# Dynamic Programming

Implemented from [Held-Karp Algorithm](https://en.wikipedia.org/wiki/Held%E2%80%93Karp_algorithm)

Time complexity: $O(n^2 2^n)$

Space complexity: $O(n 2^n)$

## Subproblem Formulation

$$ g(S,e) = \min_{1 \leq i \leq k} \{ g(S_i, s_i) + d(s_i, e) \} $$

Where: 
- $S$: Subset of nodes to be visited
  - $S \subseteq \{1, ..., n\}$
- $S_i$: Subset S with element $s_i$ removed
  - $S \backslash \{s_i\} $
- $g(S,e)$: Shortest Path from 1 to e that passes through all nodes in subset S 
  - e: Ending node
- $d(s_i, e)$: Distance from $s_i$ to e

First we need to fix the starting node. For convenience sake, let's start from node 0.
This means that all statements

Let's break the above formulation down visually:

0 -> S_e -> e

0 -> S_i -> s_i -> e



# Branch and Bound

# Conclusion

# Notes

# References
[1] []()
