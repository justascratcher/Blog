---
tags:
  - 生成函数_GF
  - 思维/基础拓展
  - Hard/思路难想
---

[C - ABS Ball](https://atcoder.jp/contests/arc212/tasks/arc212_c)

最开始看错题看成 $\sum$ 了。

考虑算出最后 $\sum|a_i-b_i|=k$ 的方案的贡献，那么就可以理解为有 $k$ 个球放在 $m$ 个盒子里，剩下的 $n-k$ 个球是红蓝一对一对出现的，这样正好没有对 $\sum$ 有贡献。那么剩下球的方案数隔板就可以算出是 $\binom{m+\frac{n-k}{2}-1}{m-1}$ .  
这里剩下要处理的问题就是 $\sum v_i=k$ 求 $\prod v_i$ ，再乘 $2^m$ 和上面的方案数就是一个 $k$ 的答案。  
主要是 $\prod v_i$ 可以理解成是在每个盒子里去取一个球的方案数，那么再利用隔板法，就相当于是在每两个板里再加一个标记，和板的标记是交替出现的，那么也就相当于是两倍的板的数量，那么答案就是 $m+k-1\choose 2m-1$.
最后，总的答案就是 
$$2^m\sum_{0\leq k \leq n, k\equiv n \pmod 2}\binom{m+\frac{n-k}{2}-1}{m-1}{m+k-1\choose 2m-1}$$ 
在网上还看到一种用GF算的方法：
显然，OGF是 $F(x)=(\sum ix^i)^m$ 
然后，
$$
\begin{aligned}
F(x)&=(\sum_{i=1}\sum_{j=i}x^j)^m\\
&=(\sum_i\frac{1}{1-x}-\frac{1-x^i}{1-x})^m\\
&=(\sum_i\frac{x^i}{1-x})^m\\
&=\left(\frac{x}{(1-x)^2}\right)^m\\
&=\frac{x^m}{(x-1)^{2m}}
\end{aligned}
$$
接下来一步他写的是极为惊人的，因为 $F(1+x)=\frac{(x+1)^m}{x^{2m}}$ ，然后GF的系数其实是 f(x) 在 0 处的泰勒展开，所以就是 $G(x)=F(x+1)$ 在 -1 处的泰勒展开。
那么
$$
\begin{aligned}
G^{(k)}(x)&=(\frac{1}{x^{2m}}\sum_i{m\choose i}x^i)^{(k)}\\
&=(\sum_i{m\choose i}x^{i-2m})^{(k)}\\
&=\sum_{i=0}^m{m\choose i}(i-2m)^\underline{k}x^{i-2m-k}\\
&=\sum_{i=0}^m{m\choose i}(2m-i+k-1)^\underline{k}(-1)^kx^{i-2m-k} \quad \textcolor{red}{???}\\
\end{aligned}
$$
接着 G(-1):
$$
\begin{aligned}
G^{(k)}(-1)&=\sum_{i=0}^m{m\choose i}(2m-i+k-1)^\underline{k}(-1)^k(-1)^{i-2m-k}\\
&=\sum_{i=0}^m{m\choose i}(2m-i+k-1)^\underline{k}(-1)^i
\end{aligned}
$$
于是
$$
\begin{aligned}
\left[x^k\right]F(x)&=\frac{F^{(k)}(0)}{i!}\\
&=\frac{G^{(k)}(-1)}{i!}\\
&=\sum_{i=0}^m{m\choose i}\frac{(2m-i+k-1)!(-1)^i}{k!(2m-i-1)!}\\
&=\sum_{i=0}^m{m\choose i}{2m-i+k-1\choose k}(-1)^i \quad \textcolor{red}{???}\\
\end{aligned}
$$
最后是看成容斥算出来
[题解：AT_arc212_c [ARC212C] ABS Ball - 洛谷专栏](https://www.luogu.com.cn/article/kkdomo8x)
但还是很晕这个方法。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define File(name) freopen(#name".in","r",stdin),freopen(#name".out","w",stdout),ios::sync_with_stdio(false),cin.tie(0),cout.tie(0)
#define For(i,a,b)for(int i = a; i <= b; i++)
#define rFor(i,a,b)for(int i=b;i>=a;i--)
#define aFor(i,v) for(auto&i:v)
#define eFor(i,u) for(int i=h[u];~i;i=ne[i])
#define ms(v,i)memset(v,i,sizeof(v))
#define l first
#define r second
#define ll long long
#define popcnt __builtin_popcount
#define ctz __builtin_ctz
#define rg(v) v.begin(),v.end()
#define int long long
const int N = 2e7+5,mod = 998244353;
int n,m;
int fac[N],inv[N];
int qpow(int a, int b){
    if(b==0)return 1;
    int v=qpow(a,b>>1);
    return v*v%mod*((b&1)?a:1)%mod;
}
void init(int s) {
    fac[0]=1;
    For(i,1,s) fac[i]=(fac[i-1]*i)%mod;
    inv[s]=qpow(fac[s],mod-2);
    rFor(i,0,s-1)inv[i]=(inv[i+1]*(i+1))%mod;
}
int C(int n,int m){
    if (n < m) return 0;
    return fac[n]*inv[n-m]%mod*inv[m]%mod;
}
signed main() {
    File(test);
    cin >> n >> m;
    init(N-1);
    int ans=0;
    // -=2 decreases Const
    for (int s=n;s>=0;s-=2) {
        int rst=(n-s)>>1;
        ans+=C(m+rst-1,m-1)*C(m+s-1,2*m-1)%mod;
        ans%=mod;
    }
    cout << qpow(2,m)*ans%mod<<endl;
    return 0;
}
```