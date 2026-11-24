---
layout: post
title: Contando triplas pitagóricas com FFT.
subtitle: Resolvendo um problema de contagem com Fast Fourier Transform.
tags: [algorithms, fft, combinatorics, geometry, number theory, c++]
author: Thiago Bastos
mathjax: true
---

Hoje quero falar sobre um problema que resolvi com FFT (Fast Fourier Transform) no Beecrowd.
O problema em questão é esse: [3187 - Apenas Mais Alguns Triângulos!](https://resources.beecrowd.com/repository/UOJ_3187.html)
que, resumidamente, dado um inteiro $N$ pede que você imprima o número de triplas $(A, B, C)$, sendo que $1 \le A, B, C \lt N$ e $A \le B$, em que $A^{2} + B^{2} \equiv C^{2} \pmod{N}$.

### Como podemos resolver?

Sabe aquele problema clássico, o Two Sum? Então, nós podemos reduzir o problema acima para ele, no entanto, vamos ter que usar FFT.

### Como reduzimos para Two Sum?

Isso é bem simples: vamos construir um polinômio $P(x) = \sum_{1}^{N - 1} x^{i^{2} \pmod {N}}$ em que cada termo corresponde a um quadrado módulo N. Mais precisamente, para cada i em [1, N - 1], adicionamos uma unidade ao coeficiente de $x^{i^{2} \pmod{N}}$.
Com isso, nós podemos encontrar $P(x)^{2}$ e depois de alguns ajustes, fornecer o resultado e para isso vamos usar FFT para encontrar a multiplicação do polinômio.
Após encontrar o resultado do quadrado do polinômio, nós vamos: somar os coeficientes do polinômio resultante, pois já encontramos as combinações de A e B dado um C fixo e iremos subtrair a quantidade de pares em que $A^{2} \equiv B^{2} \pmod{N}$ já que temos que dividir o número de combinações em dois, pois estamos usando o mesmo polinômio e $A \le B$.
Finalizando essa conta, nós podemos dividir o resultado em dois e somar o número de triplas em que os quadrados de A e B são congruentes módulo N.


### Código em C++

```c++
#include <bits/stdc++.h>

using namespace std;

using i64 = long long;
using u64 = unsigned long long;
using ld = long double;
using ii = pair<int, int>;

using cd = complex<double>;
const double PI = acos(-1);

inline int lg(int x) {
	return x ? 31 - __builtin_clz(x) : 0;
}

inline void fft(vector<cd>& a, bool invert) { 	
	const int n = static_cast<int>(a.size());

	for(int i = 1, j = 0; i < n; ++i) {
		int bit = n >> 1;	
		for(; j & bit; bit >>= 1) j -= bit;
		j += bit;
		if(i < j) swap(a[i], a[j]);
	}

    for(int len = 2; len <= n; len <<= 1) {
        double ang = 2 * PI / len * (invert ? -1 : 1);
        cd wlen(cos(ang), sin(ang));
		int mid = len >> 1;
        for(int i = 0; i < n; i += len) {
            cd w(1);
            for(int j = 0; j < mid; ++j) {
                cd u = a[i + j], v = a[i + j + mid] * w;
                a[i + j] = u + v;
                a[i + j + mid] = u - v;
                w *= wlen;
            }
        }
    }

    if(invert) for(int i = 0; i < n; ++i) a[i] /= n;
}

inline vector<i64> multiply2(vector<int>& a) {

	vector<cd> fa(a.begin(), a.end());
    
    int n = 2 << lg(2 * a.size());

	fa.resize(n);

    fft(fa, false);

    for(int i = 0; i < n; ++i) fa[i] *= fa[i];

    fft(fa, true);

	vector<i64> result(n);
    for(int i = 0; i < n; ++i) result[i] = round(fa[i].real());

	return result;
}

void solve() {
	int n;

	cin >> n;

	vector<int> P(n, 0), R(n, 0);

	for(int i = 1; i < n; ++i) {
		int X = (i64)i * i % n;
		++P[X];
		++R[2 * X % n];
	}
	
	auto Q = multiply2(P);
	i64 A = 0, B = 0;

	for(int i = 1; i < n; ++i) {
		int X = (i64)i * i % n;
		A += Q[X] + Q[X + n];
		B += R[X];
	}

	cout << (A - B) / 2 + B << '\n';
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

### Complexidade

A complexidade do algoritmo acima é $O(N log N)$, o mesmo do FFT.