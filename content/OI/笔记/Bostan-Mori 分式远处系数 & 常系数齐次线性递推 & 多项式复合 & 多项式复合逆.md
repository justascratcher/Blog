---
tags:
  - 多项式/Bostan-Mori
  - 快速数论变换_NTT
  - 常系数齐次线性递推
  - 多项式/多项式复合
  - 多项式/多项式复合逆
---
这个算法就是要求
$$
[x^k]\frac{F(x)}{G(x)}
$$
其中 F，G 都是不超过 n 次的多项式，对一个质数取模。
## 基本思想
上下同乘 G(-x) ，有
$$
\begin{aligned}
\left[x^k\right]\frac{F(x)}{G(x)}&=\left[x^k\right]\frac{F(x)G(-x)}{G(x)G(-x)}\\
\end{aligned}
$$
然后这里下面的 G(x)G(-x) 所有的奇数次项都被两两抵消没有了，只剩下偶数次项，那么可以写成 $G'(x^2)$ ，上面的式子仿照 FFT 写成奇偶两类变成 $F'_1(x^2)+xF'_2(x^2)$
接着
$$
\begin{aligned}
\left[x^k\right]\frac{F(x)}{G(x)}&=\left[x^k\right]\frac{F'_1(x^2)+xF'_2(x^2)}{G'(x^2)}\\
\end{aligned}
$$
这里其实分子上我们只关心与 k 同奇偶的次数的系数，原因是把下面式子的逆算出来后只有偶次项，与上面相乘后不改变上面系数次数的奇偶性
也就是
$$
\begin{aligned}
\left[x^k\right]\frac{F(x)}{G(x)}&=\left\{
\begin{aligned}
\left[x^k\right]\frac{F'_1(x^2)}{G'(x^2)} && k\ is \ odd\\
\left[x^k\right]\frac{xF'_2(x^2)}{G'(x^2)}&& k\ is \ even\\
\end{aligned}\right.\\
&=\left\{
\begin{aligned}
\left[x^k\right]\frac{F'_1(x^2)}{G'(x^2)} && k\ is \ odd\\
\left[x^{k-1}\right]\frac{F'_2(x^2)}{G'(x^2)}&& k\ is \ even\\
\end{aligned}\right.\\
&=\left\{
\begin{aligned}
\left[t^{k/2}\right]\frac{F'_1(t)}{G'(t)} && k\ is \ odd\\
\left[t^{(k-1)/2}\right]\frac{F'_2(t)}{G'(t)}&& k\ is \ even\\
\end{aligned}\right. \quad (t=x^2)
\end{aligned}
$$
那么就可以像 FFT 递归去计算，每次上下都先翻倍再去掉的一半的项，次数是不变的，最后 $[x^0]\frac{F}{G}=\frac{F_0}{G_0}$，复杂度 $O(n\log n\log k)$
## 代码实现
```cpp
// [x^k]F(x)/G(x)
int DivAt(Poly F, Poly G, int k) {
    while(k) {
        Poly iG=G;
        for(int i=1;i<=iG.deg;i+=2)iG[i]=-iG[i];
        F*=iG,G*=iG;
        int i=k&1; // k&1 beg important
        for(;i<=F.deg;i+=2)F[i>>1]=F[i]; 
        F.resize((i-2)>>1); // -2 important, otherwise would TLE and WA, due to the incorrect last term
        for(i=0;i<=G.deg;i+=2)G[i>>1]=G[i]; // 0 beg important
        G.resize((i-2)>>1); // -2 important
        k>>=1;
    }
    return (ll)F[0]*qpow(G[0],mod-2)%mod;
}
```
## 与常系数齐次线性递推的关系

这里假设 $a_0\cdots a_k$ 已知， $a_i=\sum_{j=0}^{k-1}c_ja_{i-k+j}\overset{j=k-j}\equiv \sum_{j=1}^{k}c_{k-j}a_{i-j}$ 的 a 的 GF 是 $h(x)=\frac{f(x)}{g(x)}$，这里的 g 是 k 次的， f 是 m 次的，那么 $f=g\times h$
那么取一个很大的 i > m，就有
$$
0=f_i=\sum_{j=0}^{k} g_jh_{i-j}
$$
把第一项抽出来（有点像杜教筛）
$$
-g_0h_i=\sum_{j=1}^{k} g_jh_{i-j}
$$
于是
$$
h_i=\sum_{j=1}^{k} \frac{-g_j}{g_0}h_{i-j}
$$
那么取 $g_0=1,g_i=-c_{k-i}\ (0<i<k)$ 就有 $h_i=\sum_{j=1}^{k} {c_{k-j}}h_{i-j}$，最后 h 的前 k 项已知，所以有 $h\equiv \frac f g \pmod{x^k}$，所以直接可以算出 $f = (g * h) \text{\ mod\ } x^k$，应用 BM 得到第 n 项系数即可。
## 代码实现
```cpp
int At(int c[],int a[],int k,int n) {
    // if (n < k) return a[n]; // optimizing code
    Poly G(k);
    G[0]=1;
    For(i,1,k)G[i]=-c[i];
    Poly R(k-1);
    For(i,0,k-1)R[i]=a[i];
    Poly F=G*R;
    F.resize(k-1); // F=(G*R)%(x^k)=G*R[0...k-1], k-1 important
    return DivAt(F,G,n);
}
```

## 完整代码
```cpp
// [x^k]F(x)/G(x)
int DivAt(Poly F, Poly G, int k) {
    while(k) {
        Poly iG=G;
        for(int i=1;i<=iG.deg;i+=2)iG[i]=-iG[i];
        F*=iG,G*=iG;
        int i=k&1; // k&1 beg important
        for(;i<=F.deg;i+=2)F[i>>1]=F[i]; 
        F.resize((i-2)>>1); // -2 important, otherwise would TLE and WA, due to the incorrect last term
        for(i=0;i<=G.deg;i+=2)G[i>>1]=G[i]; // 0 beg important
        G.resize((i-2)>>1); // -2 important
        k>>=1;
    }
    return (ll)F[0]*qpow(G[0],mod-2)%mod;
}

int At(int c[],int a[],int k,int n) {
    // if (n < k) return a[n]; // optimizing code
    Poly G(k);
    G[0]=1;
    For(i,1,k)G[i]=-c[i];
    Poly R(k-1);
    For(i,0,k-1)R[i]=a[i];
    Poly F=G*R;
    F.resize(k-1);
    return DivAt(F,G,n);
}
int n,k,c[N],a[N];
signed main() {
    Poly::init(); // init important
    File(test);
    cin>>n>>k;
    For(i,1,k)cin>>c[i],c[i]%=mod;
    For(i,0,k-1)cin>>a[i],a[i]%=mod;
    cout<<(At(c,a,k,n)%mod+mod)%mod; // output like this, important
    return 0;
}
```
[记录详情 - 洛谷 | 计算机科学教育新生态](https://www.luogu.com.cn/record/258688620)

## 多项式复合
[P5373 【模板】多项式复合函数 - 洛谷](https://www.luogu.com.cn/problem/P5373)
求 F(G(x)) 的各项系数

## 基本思路

求 F(G(x))，等价于求 $\sum G^i(x)[x^i]F(x)$

先考虑如何一次性求出 $[x^n]F(x),[x^n]F^2(x),[x^n]F^3(x),\cdots,[x^n]F^k(x)$
这件事情可以通过加一个元 y 来实现，具体地，就是把 yF(x) 作为一个整体，那么 y 的次数就是 F(x) 的次数
那么就是 
$$
\begin{align}
G(x,y)&=yF(x)+(yF(x))^2+\cdots+(yF(x))^k\\
&=\frac1{1-yF(x)}\\
\end{align}
$$
接下来类似 BM 地处理
令 $N(x,y)/M(x,y)$ 是当前的结果，这里 x 是主元，所以上下同乘 $M(-x,y)$ ，分母的奇次项没了，只剩下了偶次，于是只要视 n 的奇偶性对应保留 N(x, y)M(-x, y) 的系数换元继续递归即可
这个地方注意每次只保留小于等于当前的 n 次的项，这样子每次 x 的次数减半，y 的次数倍增，保证最后每次的数都是 O(n) 的

总复杂度 O(M(n) log n) 其中 M(n) 是算一次的复杂度，注意这里的算法是[[FFT & NTT & FWT & FMT#二维 FFT/NTT|二维NTT]]. 总复杂度 O(n log n)
## 转到本题

接着考虑如何求

参考：
[【学习笔记】Bostan-Mori 算法 - SoyTony - 博客园](https://www.cnblogs.com/SoyTony/p/Learning_Notes_about_Bostan-Mori_Algorithm.html)
[初识 Bostan-Mori 求解分式远处系数 | 题解：P4723 【模板】常系数齐次线性递推 - 洛谷专栏](https://www.luogu.com.cn/article/55ft5nsd)
[多项式不存在了：多项式复合（逆）的 O(nlog^2n) 做法 - 洛谷专栏](https://www.luogu.com.cn/article/7joh5isi)
[题解 P4723 【【模板】常系数齐次线性递推】 - 洛谷专栏](https://www.luogu.com.cn/article/oggmoj4k)
[P4723 【模板】常系数齐次线性递推 - 洛谷](https://www.luogu.com.cn/problem/solution/P4723)
[从分式第n项到线性递推 - 洛谷专栏](https://www.luogu.com.cn/article/jclscrke)
