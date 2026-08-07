---
layout: post
title: A Combinatorics Problem Solved with Lucas' Theorem.
subtitle: Case Study Solving Beecrowd's "Sibi-Xor"
tags: [algorithms, math, combinatorics, c++]
author: Thiago Felipe Bastos da Silva
mathjax: true
---

Today I wanna talk about a great mathematical theorem known as [Lucas' Theorem](https://www.topcoder.com/thrive/articles/lucas-theorem) that is used in combinatorics problems.
To illustrate it, I'll show a problem that I solved from [Beecrowd](https://judge.beecrowd.com/), the problem in this case is called [Sibi-Xor](https://resources.beecrowd.com/repository/UOJ_2811.html). Summarizing, you have to find the sum of the XOR of the AND from all subsets of elements of a given array. Moreover, the XOR must be taken from the subsets with the same number of elements, as you can see in the description of the problem that I gave to you.

But why should you use Lucas' Theorem? Because it's an elegant solution!

The solution is very straightforward, except for the theorem. You divide the problem into the contribution of each bit because it makes things easier, and for each number of elements, you have to determine whether the total number of one bits is odd. More precisely, the subsets must have a one bit set in their AND operations; furthermore, the number of one bits must be odd because of the XOR property. Now comes the great use of the theorem: since we have to check whether the number of subsets where the bitwise XOR is one is odd, you could immediately think of calculating $\binom{bits_j}{i} \pmod{2}$, but with Lucas' Theorem, you can simply use a bitwise $O(1)$ check to determine whether the number of combinations is odd.

### The algorithm

```c++
#include <bits/stdc++.h>

using namespace std;

constexpr unsigned MOD = 1e9 + 7;

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;

    cin >> n;

    auto rng = views::iota(0, n) | views::transform([](auto) {
        unsigned long long value;
        cin >> value;
        return value;
    });

    vector<int> bits(64, 0);

    ranges::for_each(rng, [&bits](auto value) {
        while(value) {
            int bit = __builtin_ctzll(value);
            ++bits[bit];
            value ^= 1llu << bit;
        }
    });

    unsigned long long answer {};

    for(int i = 1; i <= n; ++i) {
        for(int j = 0; j < 64; ++j) {
            if(bits[j] && (bits[j] & i) == i)
                answer += 1llu << j;
            answer %= MOD;
        }
    }

    cout << answer << '\n';

    return 0;
}
```