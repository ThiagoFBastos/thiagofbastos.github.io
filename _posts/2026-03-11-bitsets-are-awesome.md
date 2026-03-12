---
layout: post
title: Bitsets are Awesome
subtitle: How Bitsets can Speed Up the Time of our Solutions.
tags: [algorithms, c++, graphs, bitset]
author: Thiago Felipe Bastos da Silva
mathjax: true
---

Today I'll talk about a standard C++ data structure that is amazing: the [bitsets](https://en.cppreference.com/w/cpp/utility/bitset.html). They can handle binary operations such as left shift (<<), right shift (>>), and (&), or (|), and xor (^), and they can have a fixed number of bits.
Now you may ask me which types of problems we can solve with them, and I answer you: there are a lot, but I'll focus only on sets and the operations with them like unions and intersections.
Let's see a problem to understand it better. This problem from [AtCoder](https://atcoder.jp/) called [Ex - Directed Graph and Query](https://atcoder.jp/contests/abc287/tasks/abc287_h) seems to be interesting.
The core of this problem is: given a directed graph with $N$ vertices labeled with numbers from 1 to $N$ and given $Q$ queries, answer, for each query, what's the minimum possible maximum index over a path, if we choose the best path.
How can we answer that when $N \le 2000$, $M \le N(N - 1)$ and $Q \le 10^{4}$ in at most 4.5 s?
I could solve this problem thinking that I can solve each query $(s_i, t_i)$ by iterating over the vertex numbers in ascending order and keeping two sets: a **safe** set, which contains the vertices that I know are reachable from $s_i$, and another **unsafe** set, which contains the visited vertices that I don't know if are reachable from $s_i$. Furthermore, when I visit a vertex $v$ reachable from $s_i$, I run a BFS from it going to the vertices that are in **unsafe** and move them to the **safe** set, and I already know that the answer for these vertices is just $max(s_i, v)$, since $v$ is the maximum index that was visited (not including $s_i$).
The complexity of the algorithm below is $O(\frac{N^{2}}{64} + \frac{NQ}{64})$, which is better than the editorial, which is $O(\frac{N^3}{64}+\frac{NQ}{64})$.

### The Algorithm

```c++
#pragma GCC target("popcnt")
#include "bits/stdc++.h"

using namespace std;

#define INF   1000000000
#define INFLL 1000000000000000000ll
#define EPS 1e-9
#define all(x) x.begin(),x.end()
#define rall(x) x.rbegin(),x.rend()
#define pb push_back
#define fi first
#define sc second

using i64 = long long;
using u64 = unsigned long long;
using ld = long double;
using ii = pair<int, int>;
using i128 = __int128;

const int N = 2e3 + 10, Q = 1e4 + 10;

bitset<N> in[N], out[N], safe, unsafe;
int n, m, q, T[N], ans[Q];
vector<ii> query[N];

void solve() {
    cin >> n >> m;
    
    for(int i = 0; i < m; ++i) {
        int u, v;
        cin >> u >> v;
        --u, --v;
        out[u][v] = in[v][u] = 1;
    }

    cin >> q;

    for(int i = 0; i < q; ++i) {
        int s, t;
        cin >> s >> t;
        --s, --t;
        query[s].pb({t, i});
    }

    for(int s = 0; s < n; ++s) {
        queue<int> q;
        memset(T, -1, sizeof(int) * n);
        safe.reset();
        unsafe.reset();
        safe[s] = 1;
        T[s] = s + 1;
        for(int t = 0; t < n; ++t) {
            if(t == s) continue;
            if((safe & in[t]).count()) {    
                safe[t] = 1;
                T[t] = max(t, s) + 1;
                q.push(t);
                while(!q.empty()) {
                    int u = q.front(); q.pop();
                    auto A = out[u] & unsafe;
                    int l = A.count();
                    for(int k = 0; k < l; ++k) {
                        int v = A._Find_first();
                        unsafe[v] = 0;
                        safe[v] = 1;
                        T[v] = max(t, s) + 1;
                        q.push(v);
                        A[v] = 0;
                    }
                }
            } else unsafe[t] = 1;
        }
        for(auto [t, k] : query[s])
            ans[k] = T[t];
    }

    for(int i = 0; i < q; ++i) cout << ans[i] << '\n';
}

int main() {
    ios_base :: sync_with_stdio(false);
    cin.tie(0);
    int t = 1;
    //cin >> t;
    while(t--) solve();
    return 0;
}
```

### Tips

Some resources that I used in the algorithm above that you might not know:

- #pragma GCC target("popcnt") : this is a way to give some arguments to the GCC compiler inside the code, and this is about an instruction called **popcnt** that is used to count bits.

- _Find_first() : this is a GCC extension method for bitsets; you can learn more [here](https://codeforces.com/blog/entry/43718)