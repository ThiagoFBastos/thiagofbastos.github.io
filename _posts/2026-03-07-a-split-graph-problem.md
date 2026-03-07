---
layout: post
title: A Split Graph Problem
subtitle: Solving the Problem "Justice League" from ACM/ICPC South America Contest 2007.
tags: [algorithms, graphs, c#, prefix sum, sorting]
author: Thiago Felipe Bastos da Silva
mathjax: true
---

There is a class of graphs called [Perfect Graphs](https://mathworld.wolfram.com/PerfectGraph.html) for which some problems can be solved in polynomial time, such as: maximum clique, maximum independent set, and coloring. One such graph is the [Split Graph](https://mathworld.wolfram.com/SplitGraph.html), which can be checked by a simple $O(|V| \log |V|)$ algorithm, by simply checking if there exists at least one $m$ such that $\sum_{i = 1}^{m} d_i = m(m - 1) + \sum_{i = m + 1}^{n} d_i$, where $d_1 \le ... \le d_n$. 

You can solve a problem that involves this graph subclass, because there is a problem on [Beecrowd](https://judge.beecrowd.com) called [Justice League](https://judge.beecrowd.com/pt/problems/view/1417) from the ACM/ICPC South America Contest 2007.

### The Algorithm

```csharp
using System; 
using System.Collections.Generic;

class URI {

    struct Vertex {
        public int v;
        public int degree;
    }

    static int cmp(Vertex a, Vertex b) {
        return b.degree - a.degree;
    }

    static void Main(string[] args) {
        while(true) {

            var l = Console.ReadLine().Split(' ');

            int n = int.Parse(l[0]);
            int m = int.Parse(l[1]);

            if(n + m == 0) break;
    
            Vertex[] S = new Vertex[n];
            int[] preSum = new int[n + 1];

            for(int i = 0; i < n; ++i) {
                S[i].v = i;
                S[i].degree = 0;
            }
            
            for(int i = 0; i < m; ++i) {
                var ll = Console.ReadLine().Split(' ');
                int u = int.Parse(ll[0]) - 1, v = int.Parse(ll[1]) - 1;
                ++S[u].degree;
                ++S[v].degree;    
            }

            Array.Sort(S, cmp);

            for(int i = 1; i <= n; ++i) preSum[i] = preSum[i - 1] + S[i - 1].degree;

            bool flag = false;

            for(int i = 1; i <= n; ++i)
                flag = flag || preSum[i] == i * (i - 1) + preSum[n] - preSum[i];

            Console.WriteLine(flag ? "Y" : "N");
        }                        
    }    
}
```