---
layout: post
title: A DP Optimization Trick on Trees
subtitle: Efficient Subtree Merging in Dynamic Programming on Trees.
tags: [algorithms, tree, c++, dynamic programming, dfs]
author: Thiago Felipe Bastos da Silva
mathjax: true
---

There is a useful optimization trick involving DP on trees that I would like to discuss. It was detailed in the comments of this [Codeforces](https://codeforces.com/) [blog](https://codeforces.com/blog/entry/69888?locale=en). In short, you can define a DP over the vertices of a tree and compute its value for each $v$-subtree by combining the children of that vertex, where the DP state is a number that cannot exceed the size of the subtree.

To illustrate this idea, I will present the problem [C - Shorten Diameter](https://atcoder.jp/contests/agc001/tasks/agc001_c), which asks you to find the largest subtree whose diameter is not greater than $k$. Equivalently, you must remove the minimum possible number of vertices from the tree. This problem can be solved using a DP with states $(v, d)$, where $v$ is a vertex and $d$ represents the depth of the deepest vertex in the $v$-subtree. Furthermore, its value corresponds to the number of removed vertices, and the transitions for a vertex $v$ ensure that the sum of distances does not exceed $k$.

The final answer is simply the minimum value among all $DP(v, d)$ plus the number of vertices outside the $v$-subtree.

### The algorithm

```c++
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

const int N = 2e3 + 10;

int dp[N][N], sz[N], depth[N], tmp[N], n, k;
vector<int> adj[N];

void merge(int u, int v) {
    for(int i = 0; i <= sz[u] + sz[v]; ++i) tmp[i] = INF;
    for(int i = 0; i <= sz[u]; ++i) {
        for(int j = 0; j <= sz[v] && i + j + 1 <= k; ++j)
            tmp[max(i, j + 1)] = min(tmp[max(i, j + 1)], dp[u][i] + dp[v][j]);
        if(i <= k) tmp[i] = min(tmp[i], dp[u][i] + sz[v]);
    }
    sz[u] += sz[v];
    memcpy(dp[u], tmp, sizeof(int) * (1 + sz[u]));
}

void dfs(int u, int p) {
    sz[u] = 1;
    for(int i = 0; i <= n; ++i) dp[u][i] = INF;
    dp[u][0] = 0;
    for(int v : adj[u]) {
        if(v == p) continue;
        dfs(v, u);
        merge(u, v);
    }
}

void solve() {
    cin >> n >> k;
    for(int i = 1; i < n; ++i) {
        int a, b;
        cin >> a >> b;
        --a, --b;
        adj[a].pb(b); adj[b].pb(a);
    }
    dfs(0, -1);
    int ans = INF;    
    for(int u = 0; u < n; ++u)
        for(int i = 0; i <= k; ++i)
            ans = min(ans, dp[u][i] + n - sz[u]);
    cout << ans << '\n';
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