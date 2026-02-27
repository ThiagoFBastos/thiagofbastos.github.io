---
layout: post
title: Solving a problem with XOR hashing
subtitle: How I solved a problem from the Educational Codeforces Round 141.
tags: [algorithms, XOR hashing, c++]
author: Thiago Felipe Bastos da Silva
mathjax: true
---

Today I'll show the randomized solution for the problem [E. Game of the Year](https://codeforces.com/contest/1783/problem/E). To solve it, I used [XOR hashing](https://codeforces.com/blog/entry/85900).

The solution and the proof are very simple: the idea of the algorithm is to check, for every $k$, all intervals $(k(i - 1), ki]$ that lie in $[1, n]$ where we have both indices of the two players.


### The algorithm

```c++
#include "bits/stdc++.h"

using namespace std;

#define INF 1000000000
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

mt19937 rng((int) chrono::steady_clock::now().time_since_epoch().count());
uniform_int_distribution<i64> uid(0, 1ll << 62);

const int N = 2e5 + 100;

i64 ps[N][5];

void solve() {

    int n;

    cin >> n;

    vector<int> a(n);

    for(int i = 0; i <= n; ++i)
        for(int j = 0; j < 5; ++j)
            ps[i][j] = 0;

    for(int& v : a) cin >> v;
    for(int i = 0; i < n; ++i) {
        int b;
        cin >> b;
        if(a[i] <= b) continue;
        for(int j = 0; j < 5; ++j) {
            i64 x = uid(rng);
            ps[a[i]][j] ^= x; 
            ps[b][j] ^= x;
        }
    }

    for(int i = 1; i <= n; ++i)
        for(int j = 0; j < 5; ++j)
            ps[i][j] ^= ps[i - 1][j];

    vector<int> v;

    for(int k = 1; k <= n; ++k) {
        bool good = true;
        for(int i = 0; i < 5; ++i) {
            for(int j = 0; j <= n; j += k) {
                int l = j + 1, r = min(n, j + k);
                good = good && (ps[r][i] ^ ps[max(0, l - 1)][i]) == 0;
            }
        }
        if(good) v.pb(k);
    }
    
    cout << size(v) << '\n';
    for(int x : v) cout << x << ' ';
    cout << '\n';
}

int main() {
    ios_base :: sync_with_stdio(false);
    cin.tie(0);
    int t = 1;
    cin >> t;
    while(t--) solve();
    return 0;
}
```

### Proof

For each $1 \le k \le n$, there are two cases:

The first is when $a_j \leq b_j$: this case is very easy, since Monocarp can always kill the boss before Polycarp.

The second is when $a_j > b_j$. Thus, we have to check, for every $i \ge 1$, whether in the step that lies in the interval $(k(i - 1), ki]$, Monocarp can always kill the boss, while Polycarp would kill the boss in the next step that also lies in that interval. However, since Monocarp plays first, he can kill the boss before Polycarp.

To check this property, we only need to verify whether the XOR of all numbers in that interval is 0 (each number appears twice in that interval).