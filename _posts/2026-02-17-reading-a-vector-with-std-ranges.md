---
layout: post
title: A smarter way to read STL containers in C++.
subtitle: How std::ranges makes it easier to read collections.
tags: [programming, c++]
author: Thiago Felipe Bastos da Silva
mathjax: true
---

C++20/C++23 introduced new tools, and one of them is the ranges library, inspired by the well-known range-v3.
This standard library made it possible to work with ranges in a way never seen before. Now you can use functions such as transform, filter, drop, take, join, and many others.
All of this can be useful in Competitive Programming and in algorithms in general.

### Reading a vector in one line

Here is an example of how to read a vector in a simple way.

```c++
#include <iostream>
#include <ranges>
#include <vector>
 
using namespace std;
 
int main() {
    ios_base :: sync_with_stdio(false);
    cin.tie(0);

    int n;

    cin >> n;

    auto values = views::iota(0, n) | views::transform([](auto) {
        int val;
        cin >> val;
        return val;
    }) | ranges::to<vector<int>>();

    for(auto v : values)
        cout << v << ' ';

    cout << '\n';

    return 0;
}
```

The code above can be used to read other container types, such as std::set, std::list, std::deque, and others.