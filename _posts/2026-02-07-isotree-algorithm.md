---
layout: post
title: Isotree
subtitle: An algorithm for finding all non-isomorphic trees of a given size.
tags: [algorithms, backtracking, graphs, parallel algorithms]
author: Thiago Felipe Bastos da Silva
mathjax: true
gh-repo: ThiagoFBastos/isotree
gh-badge: [star, fork, follow]
---

After using the Prüfer code in my Scientific Initiation project to find all non-isomorphic trees of a given size, I came up with an idea to improve this algorithm, since it was slow and I could not generate the trees-or even count them for larger values of n. Therefore, I proposed a backtracking algorithm with pruning to reduce the computation time.

This backtracking algorithm is a constructive approach that builds the tree level by level while enforcing specific rules to reduce the search space. Moreover, to avoid generating duplicate trees, we used a hash table from the GNU C++ library, gp_hash_table, which is well known in the competitive programming community for being faster than std::unordered_set.

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

### A look over the algorithm

```c++
uint64_t treePat(int g[][N], int deg[], int v, int p) {
	uint64_t s[N], repr = 1;
	int len = 0;
	for(int k = 0; k < deg[v]; ++k) {
		int u = g[v][k];
		if(p == u) continue;
		s[len++] = treePat(g, deg, u, v);
	}
	sort(s, s + len);
	for(int k = 0; k < len; ++k) repr = concat(repr, s[k]);
	return concat(repr, 0);
}

uint64_t treePat(int g[][N], int deg[]) {
	auto [c1, c2] = center(g, deg);
	uint64_t S1 = treePat(g, deg, c1), S2 = 0;
	if(c2 != n) S2 = treePat(g, deg, c2);
	return max(S1, S2);
}

uint64_t concat(uint64_t a, uint64_t b) {
	int len = b ? 64 - __builtin_clzll(b) : 1;
	return (a << len) | b;
}

pair<int, int> center(int g[][N], int deg[]) {
	int T = -1, t[N], d[N], q[N], C[] = {n, n}, lo = 0, hi = 0;
	for(int v = 0; v < n; ++v) {
		d[v] = deg[v];
		t[v] = 0;
		if(deg[v] == 1) q[hi++] = v;
	}
	while(lo < hi) {
		int v = q[lo++];
		if(T <= t[v]) {
			if(T < t[v]) C[1] = n;
			C[T == t[v]] = v;
			T = t[v];
		}
		for(int k = 0; k < deg[v]; ++k) {
			int u = g[v][k];
			if(--d[u] == 1) {
				t[u] = t[v] + 1;
				q[hi++]=u;
			}
		}
	}
	return {C[0],C[1]};
}

bool prunning(int h) {
	int max_depth[N];
	for(int i = h; i >= 0; --i) {
		for(int k = lv_sz[i] - 1; k >= 0; --k) {
			int v = lv[i][k], p = par[v];
			max_depth[v] = i;
			if(deg[v] - (p != -1)) {
				int u = adj[v][p != -1];
				max_depth[v] = max_depth[u];
			}
			if(k < lv_sz[i]-1 && par[lv[i][k+1]] == p && max_depth[v]<max_depth[lv[i][k+1]]) 
				return true;
		}
	}
	return false;
}

void backtrack(int h, int cur_p, int v) {
	if(prunning(h)) return;

	if(v == -1) {
		uint64_t pat = treePat(adj, deg);
		trees.insert(pat);
		return;
	}

	int p = lv[h - 1][cur_p];

	par[v] = p;
	adj[p][deg[p]++] = v, adj[v][deg[v]++] = p;
	lv[h][lv_sz[h]++] = v;

	sub[v] = h == 1 ? n - v - 2 : sub[p];

	if(sub_lv_sz[h][sub[v]]++ == 0) ++nsub[h];

	backtrack(h, cur_p, v - 1);

	if(v && nsub[h] >= 2) backtrack(h + 1, 0, v - 1);

	if(--sub_lv_sz[h][sub[v]] == 0) --nsub[h];

	--deg[p], --deg[v], --lv_sz[h];

	if(cur_p + 1 < lv_sz[h - 1]) {
		int a = lv[h-1][cur_p], b = lv[h-1][cur_p+1], k = cur_p;
		while(k < lv_sz[h-1] && par[a] == par[lv[h-1][k]]) ++k;
		if(par[a]!=par[b] || deg[a] > 1) backtrack(h, cur_p + 1, v);
		else if(k < lv_sz[h-1]) backtrack(h, k, v);
	}
}
```

### Project

Tne project can be found in [link](https://github.com/ThiagoFBastos/isotree)
