---
layout: post
title: When i used Stirling Numbers without knowing them.
subtitle: The day I solved a combinatorial graph problem.
tags: [algorithms, dynamic programming, combinatorics, graphs]
author: Thiago Felipe Bastos da Silva
mathjax: true
---

There is a problem in [CSES](https://cses.fi/) known as [Functional Graph Distribution](https://cses.fi/problemset/task/2415) that involves graphs and combinatorics. I solved this problem, but it was tough. I didn't solve it at first sight, but one day I managed to solve it, and this story is about that day. To solve the problem, I ended up using Stirling numbers without knowing them. I thought it was just a difficult solution based on dynamic programming and combinatorics, but after seeing other people's solutions and learning what Stirling numbers were, I found out what I had actually done.

### The solution

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
 
const int N = 5e3 + 10, MOD = 1e9 + 7;
 
int f[N][N], c[N][N], dp[N], pt[N];
 
int main(int argc, char* argv[]) {
	ios_base :: sync_with_stdio(false);
	cin.tie(0);
 
	int n;
 
	cin >> n;
 
	f[0][0] = c[0][0] = pt[0] = 1;
 
	for(int i = 1; i <= n; ++i) pt[i] = (i64)n * pt[i - 1] % MOD;
 
	for(int i = 1; i <= n; ++i) {
		c[i][0] = c[i][i] = 1;
		for(int j = 1; j < i; ++j) {
			c[i][j] = c[i - 1][j] + c[i - 1][j - 1];
			if(c[i][j] >= MOD) c[i][j] -= MOD;
		}
		for(int j = 1; j <= n; ++j) 
			f[i][j] = ((j - 1ll) * f[i][j - 1] + f[i - 1][j - 1]) % MOD;
	}
 
	for(int i = n; i; --i)
		for(int j = 1; j <= n; ++j) 
			dp[i] = (dp[i] + (i64)f[i][j] * c[n][j] % MOD * pt[n - j]) % MOD;
	
	for(int i = n; i; --i) {
		dp[i] = (dp[i] + MOD) % MOD;
		for(int j = i - 1; j; --j)
			dp[j] = (dp[j] - (i64)dp[i] * c[i][j]) % MOD;
	}
 
	for(int i = 1; i <= n; ++i) cout << dp[i] << '\n';
 
	return 0;
}
```

### The proof

The purpose of the solution above is to count cycles with some nodes of permutation of $1, \ldots, n$, and then attach trees to the nodes of the cycles. For that, we find the values of the function $f(i, j)$ (this is the Stirling Number), where $i$ is the number of cycles and $j$ is the number of nodes. These values can be found using dynamic programming with the following transitions:

$f(i, j) = ((j - 1) * f(i, j - 1) + f(i - 1, j - 1)) (mod 10^{9} + 7)$

which means: given that we are adding the $j$-th node, it can be alone in a new cycle or can belong to one of the $i$ cycles; thus, it can have a direct edge to some of the $(j - 1)$ nodes.

Now, we need to find the count of functional graph components, and some nodes do not belong to any cycle; thus, these nodes are in trees. For that, we use this formula:

$dp(i) = (dp(i) + f(i, j) * \binom{n}{j} * n^{n - j}) mod (10^{9} + 7)$ $(\forall 1 \le j \le n)$

We choose a subset of $j$ nodes from the $n$ nodes and, for the remaining nodes, we add an edge to some other node. However, there is a problem: this can lead to more components, which we do not want.

Here we use the inclusion–exclusion principle to avoid repetitions that can appear in the last step. Therefore, if we compute the values of $dp(i)$ in descending order of $i$ and remove the repetitions for $j < i$, then at the current step $i$ the value is correct.

With this formula we can remove the repetitions:

$dp(j) = (dp(j) - dp(i) * \binom{i}{j}) (mod 10^{9} + 7)$

We remove $dp(i) * \binom{i}{j}$ from $dp(j)$ because any subset of $j$ components among the $i$ components can appear in $dp(j)$.