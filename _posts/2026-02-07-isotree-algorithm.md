---
layout: post
title: Isotree
subtitle: An Algorithm for Finding All Non-Isomorphic Trees of a Given Size.
tags: [algorithms, backtracking, graphs, parallel algorithms, c++, trees, ahu, isomorphism]
author: Thiago Felipe Bastos da Silva
mathjax: true
gh-repo: ThiagoFBastos/isotree
gh-badge: [star, fork, follow]
---

After using the Prüfer code in my Scientific Initiation project to find all non-isomorphic trees of a given size, I came up with an idea to improve this algorithm, since it was slow and I could not generate the trees-or even count them for larger values of n. Therefore, I proposed a backtracking algorithm with pruning to reduce the computation time.

This backtracking algorithm is a constructive approach that builds the tree level by level while enforcing specific rules to reduce the search space. Moreover, to avoid generating duplicate trees, we used a hash table from the GNU C++ library, gp_hash_table, which is well known in the competitive programming community for being faster than std::unordered_set.

One might question whether the algorithm could consume excessive memory, given that the number of trees grows rapidly and the method requires $O(n)$ space. However, this issue is mitigated by the fact that the trees considered are relatively small (since the number of non-isomorphic trees increases very quickly with n), allowing us to encode and store the trees efficiently as integers.

### Pruning rules used by the algorithm

Why do we need pruning? You might ask that.

According to Cayley’s formula, there are $n^{n - 2}$ labeled trees of a given size $n$. Because of this, with a brute-force algorithm I couldn’t compute these trees in a reasonable amount of time for larger values of $n$.

Consider small values of $n$; for example, when $n = 10$, there are $10^{8}$ labeled trees. This is a lot.

1. The tree is rooted.
2. Vertices closer to the root have greater labels than vertices farther from it.
3. For two vertices at the same level, the vertex with the larger label must have all its children labeled with values greater than those of the other vertex.
4. For two sibling vertices, the one with the larger label must have greater height.
5. The root belongs to the center of the tree; therefore, the height difference between the two tallest subtrees is at most one.


### Parallel version

A parallel version of this algorithm was implemented using OpenMP. This approach is straightforward, since each thread independently runs the serial algorithm on a distinct subtree of the root.

### Achievements

Using both the serial and parallel versions of the algorithm, we can enumerate all trees with $n \le 22$ in a reasonable amount of time. This corresponds to 5,623,756 trees. Moreover, the algorithm is capable of handling trees with up to 24 vertices, although this was not experimentally verified due to time constraints.

### The Serial Algorithm

```c++
#include <cstdio>
#include <cerrno>
#include <algorithm>
#include <iostream>
#include <utility>
#include <ext/pb_ds/assoc_container.hpp>
#include <cstring>
#include <fstream>
#include <cstdint>

using namespace std;
using namespace __gnu_pbds;

template <class K, class V>
using ht = gp_hash_table
<K, V, hash<K>, equal_to<K>,
direct_mask_range_hashing<>,
linear_probe_fn<>,
hash_standard_resize_policy<
hash_exponential_size_policy<>,
hash_load_check_resize_trigger
<>, true>>;

const int N = 30;

int lv[N][N], lv_sz[N];
int n, sub[N], deg[N], adj[N][N], par[N], sub_lv_sz[N][N], sub_sz[N], nsub[N];
ht<uint64_t, null_type> trees;

uint64_t treePat(int [][N], int[], int, int = -1);
uint64_t treePat(int [][N], int[]);
inline uint64_t concat(uint64_t, uint64_t);
pair<int, int> center(int [][N], int []);
bool prunning(int);
void backtrack(int, int, int);

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

int main(int argc, char* argv[]) {
    
    int save;
    
    if(argc < 3 || sscanf(argv[1], "%d", &n) <= 0 || sscanf(argv[2], "%d", &save) <= 0) {
        cout << "use: ./trees <nodes> <save=0/1>\n";
        return 0;
    } else if(n > 24 || n < 4) {
        cout << "the number of nodes must be between 4 and 24\n";
        return 0;
    } else if(save && argc < 4) {
        cout << "use: ./trees <nodes> 1 <file>\n";
        return 0;
    }

    lv[0][0] = n - 1, lv_sz[0] = 1;
    par[n - 1] = -1;

    backtrack(1, 0, n - 2);

    if(save) {
        ofstream fs(argv[3], ofstream :: binary);
        vector<uint64_t> t(trees.begin(), trees.end());
        int cnt_trees = trees.size();
        fs.write((const char*)&n, sizeof(int));
        fs.write((const char*)&cnt_trees, sizeof(int));
        fs.write((const char*)t.data(), sizeof(uint64_t) * t.size());
        fs.close();
    }

    cout << "were found " << trees.size() << " unlabeled trees\n";

    return 0;
}
```

### The Parallel Algorithm

```c++
#include <cstdio>
#include <cerrno>
#include <algorithm>
#include <iostream>
#include <utility>
#include <ext/pb_ds/assoc_container.hpp>
#include <cstring>
#include <fstream>
#include <cstdint>
#include <omp.h>
#include <cstdlib>
#include <cmath>

using namespace std;
using namespace __gnu_pbds;

template <class K, class V>
using ht = gp_hash_table
<K, V, hash<K>, equal_to<K>,
direct_mask_range_hashing<>,
linear_probe_fn<>,
hash_standard_resize_policy<
hash_exponential_size_policy<>,
hash_load_check_resize_trigger
<>, true>>;

const int N = 30;

struct ThreadData {
    int lv[N][N];
    int lv_sz[N];
    int sub[N];
    int deg[N];
    int adj[N][N];
    int par[N];
    int sub_lv_sz[N][N];
    int nsub[N];
    ht<uint64_t, null_type> trees;
    ThreadData() {
        memset(deg, 0, sizeof deg);
        memset(nsub, 0, sizeof nsub);
        memset(lv_sz, 0, sizeof lv_sz);
        for(int i = 0; i < N; ++i)
            memset(sub_lv_sz[i], 0, sizeof sub_lv_sz[i]);
    }
};

int n;

uint64_t treePat(int [][N], int[], int, int = -1);
uint64_t treePat(int [][N], int[]);
inline uint64_t concat(uint64_t, uint64_t);
pair<int, int> center(int [][N], int []);
bool prunning(int, int);
void backtrack(int, int, int, int);

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
    return {C[0], C[1]};
}

bool prunning(ThreadData& td, int h) {
    int max_depth[N];
    for(int i = h; i; --i) {
        for(int k = td.lv_sz[i] - 1; k >= 0; --k) {
            int v = td.lv[i][k], p = td.par[v];
            max_depth[v] = i;
            if(td.deg[v] - (p != -1)) max_depth[v] = max_depth[td.adj[v][p != -1]];
            if(k < td.lv_sz[i] - 1 && td.par[td.lv[i][k + 1]] == p && max_depth[v] < max_depth[td.lv[i][k + 1]])
                return true;
        }
    }
    return false;
}

void backtrack(ThreadData& td, int h, int cur_p, int v) {
    if(prunning(td, h)) return;

    if(v == -1) {
        uint64_t pat = treePat(td.adj, td.deg);
        td.trees.insert(pat);
        return;
    }

    int p = td.lv[h - 1][cur_p];

    td.par[v] = p;
    td.adj[p][td.deg[p]++] = v;
    td.adj[v][td.deg[v]++] = p;
    td.lv[h][td.lv_sz[h]++] = v;

    td.sub[v] = h == 1 ? n - v - 2 : td.sub[p];

    if(td.sub_lv_sz[h][td.sub[v]]++ == 0) ++td.nsub[h];

    backtrack(td, h, cur_p, v - 1);

    if(v && td.nsub[h] >= 2) backtrack(td, h + 1, 0, v - 1);

    if(--td.sub_lv_sz[h][td.sub[v]] == 0) --td.nsub[h];

    --td.deg[p];
    --td.deg[v];
    --td.lv_sz[h];

    if(cur_p + 1 < td.lv_sz[h - 1]) {
        int a = td.lv[h-1][cur_p], b = td.lv[h - 1][cur_p + 1], k = cur_p;
        while(k < td.lv_sz[h - 1] && td.par[a] == td.par[td.lv[h - 1][k]]) ++k;
        if(td.par[a] != td.par[b] || td.deg[a] > 1) backtrack(td, h, cur_p + 1, v);
        else if(k < td.lv_sz[h - 1]) backtrack(td, h, k, v);
    }
}

int main(int argc, char* argv[]) {

    int num_threads, save;

    if(argc < 4 || sscanf(argv[1], "%d", &n) <= 0 || sscanf(argv[2], "%d", &num_threads) <= 0 || sscanf(argv[3], "%d", &save) <= 0) {
        cout << "use: ./trees <nodes> <number of threads> <save=0/1>\n";
        return 0;
    } else if(n > 24 || n < 4) {
        cout << "the number of nodes must be between 4 and 24\n";
        return 0;
    } else if(save && argc < 5) {
        cout << "use: ./trees <nodes> <number of threads> 1 <file>\n";
        return 0;
    }

    omp_set_num_threads(num_threads);

    auto tdata = new ThreadData[num_threads];

    #pragma omp parallel for schedule(dynamic)
    for(int k = 1; k < n; ++k) {
        int v = n - 2, id = omp_get_thread_num();
        auto& td = tdata[id];

        td.lv[0][0] = n - 1;
        td.lv_sz[0] = 1;
        td.par[n - 1] = -1;
        td.deg[n - 1] = 0;
        td.lv_sz[1] = 0;
        td.nsub[1] = k;
        memset(td.deg, 0, sizeof td.deg);

        for(int i = 0; i < k; ++i, --v) {
            td.par[v] = n - 1;
            td.adj[n - 1][td.deg[n - 1]++] = v;
            td.adj[v][td.deg[v]++] = n - 1;
            td.lv[1][td.lv_sz[1]++] = v;
            td.sub[v] = n - v - 1;
            td.sub_lv_sz[1][n - v - 1]=1;
        }

        backtrack(td, 2, 0, v);
    }

    vector<uint64_t> t;

    #ifdef DEBUG
    double median = 0, dsp = 0;
    for(int i = 0; i < num_threads; ++i) median += tdata[i].trees.size();
    median /= num_threads;
    #endif

    for(int i = 0; i < num_threads; ++i) {
        auto& td = tdata[i];
        auto& tree = td.trees;
        t.insert(t.end(), tree.begin(), tree.end());
        #ifdef DEBUG
        double val = tree.size() - median;
        dsp += val * val;
        #endif
    }

    sort(t.begin(), t.end());
    t.resize(unique(t.begin(), t.end()) - t.begin());

    if(save) {
        ofstream fs(argv[4], ofstream :: binary);
        int cnt_trees = t.size();
        fs.write((const char*)&n, sizeof(int));
        fs.write((const char*)&cnt_trees, sizeof(int));
        fs.write((const char*)t.data(), sizeof(uint64_t) * t.size());
        fs.close();
    }

    cout << "were found " << t.size() << " unlabeled trees\n";

    #ifdef DEBUG
    cout.precision(3);
    cout.setf(ios_base :: fixed);
    cout << "standard deviation between the count of trees over the threads: " << sqrt(dsp / num_threads) << '\n';
    #endif

    delete[] tdata;

    return 0;
}
```

### Project

Tne project can be found in [link](https://github.com/ThiagoFBastos/isotree)
