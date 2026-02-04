---
tags:
  - 快速傅里叶变换_FFT
  - 快速数论变换_NTT
  - undone
  - 二维NTT
---
快速测试链接：
[P3803 【模板】多项式乘法（FFT） - 洛谷](https://www.luogu.com.cn/problem/P3803)
[P4717 【模板】快速莫比乌斯 / 沃尔什变换 (FMT / FWT) - 洛谷](https://www.luogu.com.cn/problem/P4717)

## 基本思想

把多项式从系数表示转到点值表示，完成相乘后再转回系数表示。
原理：
令 $f(x)=\sum_i a_ix^i, g(x)=\sum_ib_ix^i$
如果 $h(x)=f(x)\cdot g(x)$ ，那么 $h(x) = \sum_i (\sum_{j=0}^i a_jb_{i-j})x^i$，直接计算系数复杂度是 $O(n^2)$ 的，
但是如果取几个特定的点来表示 f、 g 就可以把 f、 g 写成 $\{\{x_0,f(x_0)\},\{x_1,f(x_1)\}\dots\}$，
然后做乘法就是$\{\{x_0,f(x_0)g(x_0)\},\{x_1,f(x_1)g(x_1)\}\dots\}$，复杂度是 $O(n)$ 的，
最后只要能快速在两种表示间切换就行了。

## FFT

这里的思路是把系数分为奇偶两类，然后带入单位根来导出结果。
先把系数补全到 $2^k$ 下文的 $n=2^k$。
把系数按奇偶分类，$f(x)=(a_0x^0+a_2x^2+\dots)+(a_1x^1+a_3x^3+\dots)$，
然后令 $f_1(x)=(a_0x^0+a_2x^2+\dots)$，$f_2(x)=(a_1x^1+a_3x^3+\dots)$
那么 $f(x)=f_1(x^2)+xf_2(x^2)$
那么令 n 次单位根是 $\omega_n$，那带入 $\omega_n^k\quad(k<n/2)$
$$
\begin{aligned}
f(\omega_n^k)&=f_1(\omega_n^{2k})+\omega_n^kf_2(\omega_n^{2k})\\
&=f_1(\omega^k_{n/2})+\omega_n^kf_2(\omega^k_{n/2})
\end{aligned}
$$
再代入 $\omega_n^{k+n/2}\quad(k<n/2)$
$$
\begin{aligned}
f(\omega_n^{k+n/2})&=f_1(\omega_n^{2(k+n/2)})+\omega_n^{k+n/2}f_2(\omega_n^{2(k+n/2)})\\
&=f_1(\omega_n^{2k+n})-\omega_n^kf_2(\omega_n^{2k+n})\\
&=f_1(\omega_n^{2k})-\omega_n^kf_2(\omega_n^{2k})\\
&=f_1(\omega_{n/2}^{k})-\omega_n^kf_2(\omega_{n/2}^{k})\\
\end{aligned}
$$
观察式子可以发现，要求 $f(\omega_n^{k})$， $f(\omega_n^{k+n/2})$ 只需要知道 $f_1(\omega_{n/2}^{k})$，$f_2(\omega_{n/2}^{k})$ 的值就可以了，那对于 $f_1(\omega_{n/2}^{k})$ ，就可以再去求 $f_{11}(\omega_{n/4}^{k})$ 和 $f_{12}(\omega_{n/4}^{k})$ 就可以了，这里求 $f_1(\omega_{n/2}^{k})$，$f_2(\omega_{n/2}^{k})$ ~~需要求的新 f 是一样的~~ 实际上要求的函数本身是不同的，那么其实就只要每层扩展一倍的函数的量，那么就递归求下去，直到求 $f_{\dots}(\omega_{n/(2^k)}^{k})=f_{\dots}(\omega_{1}^{k})$ 就能直接得到答案其实就是 $\{1,f(1)\}$ 然后就做完了。

## IFFT

接下来将两个函数相乘后，为了得到系数表示，还要逆着做一遍 FFT，也就是 IFFT。
这里是观察发现 DFT 其实就是求：
$$
\left[\begin{matrix}
1&1&1&\dots&1\\
1&\omega^1_n&(\omega^1_n)^2&\dots&(\omega^1_n)^{n-1}\\
1&\omega^2_n&(\omega^2_n)^2&\dots&(\omega^2_n)^{n-1}\\
\vdots&\vdots&\vdots&\ddots&\vdots\\
1&\omega^{n-1}_n&(\omega^{n-1}_n)^2&\dots&(\omega^{n-1}_n)^{n-1}\\
\end{matrix}\right]
\left[\begin{matrix}
a_0\\a_1\\a_2\\\vdots\\a_n
\end{matrix}\right]
=\left[\begin{matrix}
f(1)\\f(\omega^1_n)\\f(\omega^2_n)\\\vdots\\f(\omega^{n-1}_n)
\end{matrix}\right]
$$

那么反向求只要求出左边矩阵的逆就 ok 了。

**注意到** $x^n-1=(x-1)(x^{n-1}+x^{n-2}+\cdots+1)$，若 x 是单位根，那么 $x^n-1=0$，于是要么 $x=1$ ，要么 $x^{n-1}+x^{n-2}+\cdots+1=0$
于是再与矩阵乘法联系起来，得到
$$
\begin{aligned}
\left[
\begin{matrix}
1&\omega^{n-i}_n&(\omega^{n-i}_n)^2&\cdots&(\omega^{n-i}_n)^{n-1}
\end{matrix}
\right]
\left[
\begin{matrix}
1\\{\omega^1_n}^j\\(\omega^2_n)^j\\\vdots\\(\omega^{n-1}_n)^j
\end{matrix}
\right]
&=\sum_{k=0}^{n-1}(\omega^{n-i}_n)^k(\omega^j_n)^k\\
&=\sum_{k=0}^{n-1}(\omega^{n-i}_n\omega^j_n)^k\\
&=\sum_{k=0}^{n-1}(\omega^{n-i+j}_n)^k\\
&=\left\{\begin{align}n&&i=j\\0&&i\neq j\end{align}\right.
\end{aligned}
$$
上面这个式子后面的就是原矩阵的列，前面的是假想中的逆矩阵的第 i 行，这其实就是矩阵乘法时算的 (i, j) 处的值的过程，那么构造这样的矩阵和原矩阵相乘，所得到的刚好是单位矩阵的 n 倍，最后再除掉 n 就是逆矩阵。
于是 
$$
\frac1n\left[\begin{matrix}
1&1&1&\dots&1\\
1&\omega^{n-1}_n&(\omega^{n-1}_n)^2&\dots&(\omega^{n-1}_n)^{n-1}\\
1&\omega^{n-2}_n&(\omega^{n-2}_n)^2&\dots&(\omega^{n-2}_n)^{n-1}\\
\vdots&\vdots&\vdots&\ddots&\vdots\\
1&\omega^1_n&(\omega^1_n)^2&\dots&(\omega^1_n)^{n-1}\\
\end{matrix}\right]
$$
又
$$
\omega^{n-i}_n=\overline{\omega^{i}_n}
$$
那么
$$
\frac1n\left[\begin{matrix}
1&1&1&\dots&1\\
1&\omega^{n-1}_n&(\omega^{n-1}_n)^2&\dots&(\omega^{n-1}_n)^{n-1}\\
1&\omega^{n-2}_n&(\omega^{n-2}_n)^2&\dots&(\omega^{n-2}_n)^{n-1}\\
\vdots&\vdots&\vdots&\ddots&\vdots\\
1&\omega^1_n&(\omega^1_n)^2&\dots&(\omega^1_n)^{n-1}\\
\end{matrix}\right]=
\frac1n\left[\begin{matrix}
1&1&1&\dots&1\\
1&\overline{\omega^1_n}&(\overline{\omega^1_n})^2&\dots&(\overline{\omega^1_n})^{n-1}\\
1&\overline{\omega^2_n}&(\overline{\omega^2_n})^2&\dots&(\overline{\omega^2_n})^{n-1}\\
\vdots&\vdots&\vdots&\ddots&\vdots\\
1&\overline{\omega^{n-1}_n}&(\overline{\omega^{n-1}_n})^2&\dots&(\overline{\omega^{n-1}_n})^{n-1}\\
\end{matrix}\right]
$$
搞了半天原来求 IFFT 就直接在求 FFT 时把取单位根改成取单位根的共轭，最后除个 n 就行了。

## 代码实现
```cpp
#include <bits/stdc++.h>
using namespace std;

#define File(name) freopen(#name".in","r",stdin),freopen(#name".out","w",stdout),ios::sync_with_stdio(false),cin.tie(0),cout.tie(0)
#define For(i,a,b)for(int i = a; i <= b; i++)
#define rFor(i,a,b)for(int i=b;i>=a;i--)
#define aFor(i,v) for(auto&i:v)
#define eFor(i,u) for(int i=G::h[u];~i;i=G::ne[i])
#define erFor(i,u) for(int i=G::rh[u];~i;i=G::ne[i])
#define ms(v,i)memset(v,i,sizeof(v))
#define l first
#define r second
#define ll long long
#define pb push_back
#define popcnt __builtin_popcount
#define ctz __builtin_ctz
#define rg(v) v.begin(),v.end()
#define int long long
#define pii pair<int,int>
#define endl '\n'

const int mod=998244353, N = 1<<22; // not 2e6, due to alignment to 2^n
void madd(int&a,int b){
    a=(a+b)%mod;
}
void mmul(int&a,int b){
    a=(ll)a*b%mod;
}
int lowbit(int u) {return u&(-u);}
int qpow(int a,int b){
    if (b==0)return 1;
    int v=qpow(a,b>>1);
    return (ll)v*v%mod*((b&1)?a:1)%mod;
}

#define Comp complex<double>
double pi=3.14159265358979323846224338;
Comp F[N],G[N], tmp[N];
void FFT(Comp* f,int n,int inv) { // inv = 1 FFT, inv = -1 IFFT
    if (n == 1) return;
    int hlf=n/2;
    for(int i=0;i<n;i+=2) {
        tmp[i/2]=f[i];
        tmp[i/2+hlf]=f[i+1];
    }
    For(i,0,n-1)f[i]=tmp[i];
    FFT(f,hlf,inv),FFT(f+hlf,hlf,inv);
    Comp omg(1,0),step(cos(2*pi/n),inv*sin(2*pi/n));
    For(i,0,hlf-1){
        tmp[i]=f[i]+omg*f[i+hlf];
        tmp[i+hlf]=f[i]-omg*f[i+hlf];
        omg*=step;
    }
    For(i,0,n-1){
        f[i]=tmp[i];
        // notice: cannot div n here
    }
}

void mul(Comp* F, int n, Comp* G, int m) {
    // important, n+m is the final answer's len, 
    // less than this can cause loss of data
    int l=1<<max((int)ceil(log2(n+m+1)),1ll); 
    // must max(...,1) for reason: if one is a constant, 
    // and the other is a linear function, then 0+1=1
    // and log 1 = 0, 1<<log 1=1, nothing is done 
    // in the function.

    FFT(F,l,1);
    FFT(G,l,1);
    For(i,0,l)F[i]*=G[i];
    FFT(F,l,-1);
    For(i,0,l)F[i]/=l;
}

signed main() {    
	File(test);
    int n,m;cin>>n>>m;
    For(i,0,n)cin>>F[i];
    For(i,0,m)cin>>G[i];
    mul(F,n,G,m);
    For(i,0,n+m){
        cout<<(int)(round(F[i].real()))<<" ";
    }
    return 0;
}
```
[记录详情 - 洛谷 | 计算机科学教育新生态](https://www.luogu.com.cn/record/258360934)

参考：
[快速傅里叶变换（FFT）超详解 - 知乎](https://zhuanlan.zhihu.com/p/347091298)

## NTT

由于 FFT 要浮点运算，有误差，速度还慢，所以有了 NTT
这里要用到原根，就是对于一个 p，满足 $\delta_n(g)=\varphi(p)$，且 $\text{gcd}(g,p)=1$ 的数。
设 p 有以下良好的性质：
1. p 是质数
2. p = $a2^k+1$ 其中的 k 很大
那么这样的 p 就是极好的 NTT 模数，而且是存在的（比如经典的 998244353）
这里取一个这样的 p
设 $g_n=g^{(p-1)/n}$ ，其在模 p 意义下具有上述 $\omega_n$ 的所有性质，其中乘共轭就是乘逆元

求质数 p 的原根可以直接枚举，条件就是 $g^n$ 最小的周期是 p-1，也就是 $g^{\text{true\ factor\ of\ }p-1}\equiv 1\pmod p$ 都不成立，并且 $g^{p-1}\equiv 1\pmod p$ 成立。

做题时注意所给的模数是否能够做 NTT
## 代码实现
```cpp
#include <bits/stdc++.h>
using namespace std;
#define IOopt ios::sync_with_stdio(false),cin.tie(0),cout.tie(0)
#define File(name) freopen(#name".in","r",stdin),freopen(#name".out","w",stdout),ios::sync_with_stdio(false),cin.tie(0),cout.tie(0)
#define For(i,a,b)for(int i = a; i <= b; i++)
#define rFor(i,a,b)for(int i=b;i>=a;i--)
#define aFor(i,v) for(auto&i:v)
#define eFor(i,u) for(int i=G::h[u];~i;i=G::ne[i])
#define erFor(i,u) for(int i=G::rh[u];~i;i=G::ne[i])
#define ms(v,i)memset(v,i,sizeof(v))
#define l first
#define r second
#define ll long long
#define pb push_back
#define popcnt __builtin_popcount
#define ctz __builtin_ctz
#define rg(v) v.begin(),v.end()
#define int long long
#define pii pair<int,int>
#define endl '\n'

const int mod=998244353, N = 1<<22; // not 2e6, due to alignment to 2^n
// 998244353 is a fantastic mod
void madd(int&a,int b){
    a=(a+b)%mod;
}
void mmul(int&a,int b){
    a=(ll)a*b%mod;
}
int lowbit(int u) {return u&(-u);}
int qpow(int a,int b){
    if (b==0)return 1;
    int v=qpow(a,b>>1);
    return (ll)v*v%mod*((b&1)?a:1)%mod;
}
const int g=3, gi=qpow(3,mod-2);
int gn[24],gin[24];
int F[N],G[N],tmp[N];
// consider butterfly opt
void NTT(int* f,int n,int inv) { // inv = 1 NTT, inv = -1 INTT
    if (n == 1) return;
    int hlf=n/2;
    for(int i=0;i<n;i+=2){
        tmp[i/2]=f[i];
        tmp[i/2+hlf]=f[i+1];
    }
    For(i,0,n-1)f[i]=tmp[i];
    NTT(f,hlf,inv),NTT(f+hlf,hlf,inv);
    int omg=1,step=inv==1?gn[ctz(n)]:gin[ctz(n)];
    For(i,0,hlf-1){
        tmp[i]=(f[i]+(ll)omg*f[i+hlf])%mod;
        tmp[i+hlf]=(f[i]-(ll)omg*f[i+hlf])%mod;
        omg=((ll)omg*step)%mod;
    }
    For(i,0,n-1)f[i]=tmp[i];
}

void mul(int*F,int n,int*G,int m){
    int l=1<<max((int)ceil(log2(n+m+1)),1ll);
    NTT(F,l,1);
    NTT(G,l,1);
    For(i,0,l-1)F[i]=(F[i]*G[i])%mod;
    NTT(F,l,-1);
    int inv=qpow(l,mod-2);
    For(i,0,l-1)F[i]=(F[i]*inv%mod+mod)%mod;
}

signed main() {
    IOopt;
    int n,m;cin>>n>>m;
    gn[0]=g,gin[0]=gi;
    For(i,1,23)
        gin[i]=qpow(gi,(mod-1)/(1<<i)),
        gn[i]=qpow(g,(mod-1)/(1<<i));
    For(i,0,n)cin>>F[i];
    For(i,0,m)cin>>G[i];
    mul(F,n,G,m);
    For(i,0,n+m){
        cout<<F[i]<<" ";
    }
    return 0;
}
```
[记录详情 - 洛谷 | 计算机科学教育新生态](https://www.luogu.com.cn/record/258374375)
[记录详情 - 洛谷 | 这个没卡常](https://www.luogu.com.cn/record/258372305)

参考：
[快速数论变换（NTT）超详解 - 知乎](https://zhuanlan.zhihu.com/p/347726949)
## 蝶形运算优化

首先可以把分治算法换成倍增，就相当于是先模拟出把所有系数都放到在最后一层递归的位置，然后向回合并，那么只要能够快速求出最后的位置就行了。
可以得到第 i 个位置会到 i 的二进制位翻转后的位置，证明大概就是 i 位置的最高位会在第一轮交换被确定，然后次高位会在第二轮交换被确定，并且刚好最高位和原先的第一位是相同的，第二次交换也一样，那么就证完了。
这里设 $rev_i$ 是第 i 个系数到的位置，那么有递推式
$$
rev_i=\left\lfloor rev_{\left\lfloor i/2\right\rfloor}/2\right\rfloor+(len/2)(i\text{\ mod\ }2)
$$
其中 len 是序列长度，详细推理略
接着在回推时，本来的 f 应该回到了正确的位置，之后再去推时，因为 $f'_i=f_i+\omega_n^{i} f_{i+n/2}$，$f'_{i+n/2}=f_i-\omega_n^i f_{i+n/2}$ ，于是不用改位置，直接原地计算即可。

## 代码实现
```cpp
const int mod=998244353, N = 1<<22; // not 2e6, due to alignment to 2^n
void madd(int&a,int b){
    a=(a+b)%mod;
}
void mmul(int&a,int b){
    a=(ll)a*b%mod;
}
int lowbit(int u) {return u&(-u);}
int qpow(int a,int b){
    if (b==0)return 1;
    int v=qpow(a,b>>1);
    return (ll)v*v%mod*((b&1)?a:1)%mod;
}

namespace FFT {

#define Comp complex<double>
double pi=3.14159265358979323846224338;
Comp F[N],G[N], tmp[N];
int rev[N];
void calcRev(int l) {
    int len=ctz(l)-1;// -1 important
    rev[0]=0;
    For(i,1,l-1)rev[i]=(rev[i>>1]>>1)|(i&1)<<len;
}
void FFT(Comp* f,int n,int inv) { // inv = 1 FFT, inv = -1 IFFT
    For(i,0,n-1)
        if(i<rev[i]) swap(f[i],f[rev[i]]);
    for(int i=1;i<n;i<<=1){// here i is HALF of the range
        Comp step(cos(pi/i),inv*sin(pi/i)); // no mul 2 important
        for(int j=0;j<n;j+=i<<1){
            Comp omg(1,0);
            For(k,j,j+i-1) {
                Comp f1=f[k],f2=omg*f[k+i];
                f[k]=f1+f2,f[k+i]=f1-f2;
                omg*=step;
            }
        }
    }
}

void mul(Comp* F, int n, Comp* G, int m) {
    int l=1<<max((int)ceil(log2(n+m+1)),1ll);
    calcRev(l);
    FFT(F,l,1);
    FFT(G,l,1);
    For(i,0,l)F[i]*=G[i];
    FFT(F,l,-1);
    For(i,0,l)F[i]/=l;
}

}

namespace NTT {

const int mod = 998244353, g = 3, gi = qpow(g, mod - 2);
int F[N], G[N],gn[24],gin[24];
int rev[N];
void calcRev(int l) {
    int len=ctz(l)-1;// -1 important
    rev[0]=0;
    For(i,1,l-1) rev[i]=(rev[i>>1]>>1)|((i&1)<<len);
}
void init(){
    For(i,0,23)gn[i]=qpow(g,(mod-1)>>i),gin[i]=qpow(gi,(mod-1)>>i);
}
void NTT(int* f,int n, int inv) { // inv = 1 NTT, inv = -1 INTT
    For(i, 0, n-1) if (i<rev[i]) swap(f[i],f[rev[i]]);
    for(int i=1;i<n;i<<=1) {// i is HALF of length of the range
        int step= inv == 1 ? gn[ctz(i<<1)] : gin[ctz(i<<1)]; // <<1 important
        for(int j=0;j<n;j+=i<<1){
            int omg=1;
            For(k,j,j+i-1){
                int f1=f[k],f2=(ll)f[k+i]*omg%mod;
                f[k]=(f1+f2)%mod;
                f[k+i]=(f1-f2)%mod;
                omg=(ll)omg*step%mod;
            }
        }
    }
}
void mul(int*F,int n,int*G,int m){
    init();
    int l=1<<max(1ll,(int)ceil(log2(m+n+1)));
    calcRev(l);
    NTT(F,l,1),NTT(G,l,1);
    For(i,0,l-1)F[i]=((ll)F[i]*G[i])%mod;
    NTT(F,l,-1);
    int inv=qpow(l,mod-2);
    For(i,0,l-1)F[i]=(ll)F[i]*inv%mod;
    For(i,0,l-1)F[i]=(F[i]+mod)%mod;
}

}

signed main() {
    File(test);
    int n,m;cin>>n>>m;
    For(i,0,n)cin>>NTT::F[i];
    For(i,0,m)cin>>NTT::G[i];
    NTT::mul(NTT::F,n,NTT::G,m);
    For(i,0,n+m){
        cout<<NTT::F[i]<<" ";
    }
    return 0;
}
```

参考：
[快速傅里叶变换 - OI Wiki](https://oi-wiki.org/math/poly/fft/#%E5%80%8D%E5%A2%9E%E6%B3%95%E5%AE%9E%E7%8E%B0)
[快速傅里叶变换（FFT）的优化 - 知乎](https://zhuanlan.zhihu.com/p/347514286)


## Poly 的板子

```cpp
struct mint {
    int v;
    mint(int v=0):v(v){}
    mint operator+(mint b){return (v+b.v)%mod;}
    mint operator-(mint b){return (v-b.v)%mod;}
    mint operator-(){return -v;}
    mint operator*(mint b){return (ll)v*b.v%mod;}
    mint operator/(mint b){return (ll)v*qpow(b.v,mod-2)%mod;}
    mint& operator +=(mint b){return (*this)=(*this+b);}
    mint& operator -=(mint b){return (*this)=(*this-b);}
    mint& operator *=(mint b){return (*this)=(*this*b);}
    mint& operator /=(mint b){return (*this)=(*this/b);}
    bool operator ==(mint b){return v==b.v;}
    bool operator !=(mint b){return v!=b.v;}
    bool operator >=(mint b){return v>=b.v;}
    bool operator <=(mint b){return v<=b.v;}
    bool operator >(mint b){return v>b.v;}
    bool operator <(mint b){return v<b.v;}
    mint operator >>(mint b){return v>>b.v;}
    mint operator <<(mint b){return v<<b.v;}    
    friend ostream& operator<<(ostream&os,mint v){
	    return os<<v.reg();
    }
    friend istream& operator>>(istream&os,mint v){
	    return os>>v.v;
    }
    int reg()const{
        return (v+mod)%mod;
    }
    explicit operator int() {
        return v;
    }
};

mint qpow(mint a,mint b){
    if (b==0)return 1;
    mint v=qpow(a,b>>1);
    return v*v*((int(b)&1)?a:1);
}

int inv[N],siz[N];
static int g,gi,gn[24],gin[24];
static int rev[N];
static void init() {
    g=3;gi=qpow(g,mod-2);
    For(i,0,23)gn[i]=qpow(g,(mod-1)>>i),gin[i]=qpow(gi,(mod-1)>>i);
    inv[1]=1;
    For(i,2,N)inv[i]=-(ll)inv[mod%i]*(mod/i)%mod;
    siz[0]=1;
    For(i,1,N-1)siz[i]=siz[i/2]<<1;
}
int cnt=0;
template<typename T>
struct Poly {
    vector<T> c;
    int deg;
    explicit Poly(int n=0){
        resize(n);
    }
    ~Poly() {
        vector<T>().swap(c);
    }
    Poly& resize(int n) {
        c.resize(n+1);
        c.resize(siz[n+1]);
        deg=n;
        return *this;
    }
    void calcRev(int l){
        static int lst=-1;
        cnt++;
        lst=l;
        int len=ctz(l)-1;
        rev[0]=0;
        For(i,1,l-1){
            rev[i]=(rev[i>>1]>>1)|((i&1)<<len);
        }
    }
    Poly& NTT(int inv) { // inv = 1 NTT, inv = -1 INTT
        int l=c.size();
        calcRev(l);
        For(i,0,l-1)if(i<rev[i])swap(c[i],c[rev[i]]);
        for(int i=1;i<l;i<<=1) { // i is HALF the range len
            const mint step= inv == 1 ? gn[ctz(i<<1)] : gin[ctz(i<<1)]; // << 1 important
            for(int j=0;j<l;j+=i<<1) { // << 1 important
                mint omg = 1;
                For(k,j,j+i-1){
                    T f1=c[k], f2=omg*c[k+i];
                    c[k]=(f1+f2);
                    c[k+i]=(f1-f2);
                    omg=omg*step;
                }
            }
        }
        return *this;
    }
    T& operator[](size_t i){
        return c[i];
    }
    const T& operator[](size_t i) const {
        return c[i];
    }
    Poly& operator*=(Poly G) {
        Poly& F=*this;
        int deg=F.deg+G.deg;
        F.resize(deg);
        G.resize(deg);
        int l=F.c.size();
        calcRev(l);
        F.NTT(1),G.NTT(1);
        For(i,0,l-1)F[i]=F[i]*G[i];
        F.NTT(-1);
        int inv=qpow(l,mod-2);
        For(i,0,l-1)F[i]=F[i]*inv;
        return *this;
    }
    friend Poly operator*(Poly G, Poly F) {
        return F*=G;
    }

    Poly operator*=(mint v) {
        For(i,0,deg)c[i]=c[i]*v;
        return *this;
    }
    friend Poly operator*(mint v, Poly F) {
        return F*=v;
    }
    friend Poly operator*(Poly F, mint v) {
        return F*=v;
    }
    Poly& operator+=(Poly G) {
        Poly& F=*this;
        F.resize(max(F.deg,G.deg));
        For(i,0,G.deg)F[i]=F[i]+G[i];
        return *this;
    }
    Poly& operator-=(Poly G) {
        Poly& F=*this;
        F.resize(max(F.deg,G.deg));
        For(i,0,G.deg)F[i]=F[i]-G[i];
        return *this;
    }
    
    friend Poly operator+(Poly F, Poly G) {
        return F+=G;
    }
    friend Poly operator-(Poly F, Poly G) {
        return F-=G;
    }

    Poly operator-() {
        Poly& F=*this;Poly G(F.deg);
        For(i,0,G.deg)G[i]=-F[i];
        return G;
    }

    Poly D() {
        Poly& F=*this;
        if (F.deg==0)return Poly(0);
        Poly G(F.deg-1);
        For(i,0,G.deg) G[i]=(ll)(i+1)*F[i+1]%mod;
        return G;
    }

    Poly I() {
        Poly& F=*this;
        Poly G(F.deg+1);G[0]=0;
        For(i,1,G.deg) G[i]=(ll)F[i-1]*inv[i]%mod;
        return G;
    }

    friend std::ostream& operator<<(std::ostream& os, const Poly& F) {
        For(i,0,F.deg) os<<F[i]<<" "; return os;
    }
    friend std::istream& operator>>(std::istream& os, Poly& F) {
        For(i,0,F.deg) os>>F[i]; return os;
    }
};
using poly = Poly<mint>;
```


## 二维 FFT/NTT

[U107257 【模板】二维 FFT - 洛谷](https://www.luogu.com.cn/problem/U107257)
$H(x,y)=F(x,y)G(x,y)$ ，F G 系数已知，求 H 的各项系数.
## 方法

只要在前面的基础上把系数写成矩阵，先对行做 NTT 再对列做 NTT，乘完后再对行做 INTT 然后对列做 INTT 就 ok 了，这里先行还是先列并不重要。原理是因为 NTT 在两个维度上是独立的（[[二维 FFT NTT|Deepseek 的解答]]）
## 代码实现
```cpp
const int mod=998244353, N = 1<<22; 
constexpr int qpow(int a,int b){
    if (b==0)return 1;
    int v=qpow(a,b>>1);
    return (ll)v*v%mod*((b&1)?a:1)%mod;
}
const int g=3,gi=qpow(g,mod-2);
int gn[22],gin[22];
static int rev[N];
struct poly2 {
    vector<vector<int>> c;
    int d1,d2;
    static void init() {
        For(i,0,21)gn[i]=qpow(g,(mod-1)>>i);
        For(i,0,21)gin[i]=qpow(gi,(mod-1)>>i);
    }

    poly2(int n,int m){resize(n,m);}

    void resize(int n,int m){
        c.resize(n+1);
        c.resize(1<<max(lg(n)+1,1));
        int siz=1<<max(lg(m)+1,1);
        aFor(v,c)v.resize(m),v.resize(siz);
        d1=n,d2=m;
    }
    void calcRev(int l){
        int k=ctz(l)-1;
        rev[0]=0;
        For(i,1,l-1)rev[i]=rev[i>>1]>>1|((i&1)<<k);
    }
    void NTTC(int inv) { // inv = 1 NTT, inv = -1 INTT
        int l=c[0].size();
        calcRev(l);
        aFor(v,c){
            For(i,0,l-1)if(i<rev[i])swap(v[i],v[rev[i]]);
            for(int i=1;i<l;i<<=1){ // i is half the range len
                int step = inv == 1 ? gn[ctz(i << 1)] : gin[ctz(i << 1)];
                for(int j=0;j<l;j+=i<<1){
                    int omg = 1;
                    For(k,j,j+i-1){
                        int f1 = v[k], f2 = (ll)omg * v[k+i] % mod;
                        v[k] = (f1+f2)%mod;
                        v[k+i] = (f1-f2)%mod;
                        omg = (ll)omg * step % mod;
                    }
                }
            }
            if (inv == -1) {
                int iv = qpow(l, mod - 2);
                For(i,0,l-1){
                    v[i] = (ll)v[i] * iv % mod;
                }
            }
        }
    }
    void NTT(int inv) {
        NTTC(inv),transpose(),
        NTTC(inv),transpose();
    }
    void transpose() {
        vector<vector<int>> nc(c[0].size(), vector<int>(c.size()));
        For(i,0,c.size()-1)For(j,0,c[0].size()-1){
            nc[j][i]=c[i][j];
        }
        swap(nc, c);
    }

    vector<int>& operator[](size_t i) {
        return c[i];
    }

    poly2& operator *= (poly2 G) {
        poly2&F=*this;
        int d1=F.d1+G.d2, d2=F.d2+G.d2;
        F.resize(d1,d2),F.NTT(1);
        G.resize(d1,d2),G.NTT(1);
        For(i,0,F.c.size()-1)For(j,0,F[0].size()-1)F[i][j]=(ll)F[i][j] * G[i][j] % mod;
        F.NTT(-1);
        For(i,0,F.c.size()-1)For(j,0,F[0].size()-1)F[i][j]=(F[i][j]%mod+mod)%mod;
        return *this;
    }
};
```
[记录详情 - 洛谷 | 计算机科学教育新生态](https://www.luogu.com.cn/record/259898682)
## 相关练习
[[ABC439G - Sugoroku 6]]

## FWT

在 FFT 的基础上继续思考会发现，对于一个问题，只要有快速的变换和逆变换，并且在变换后有快速的算法计算，那就可以快速解决
FWT就可以用来求解二进制位运算相关的求和
事实上，FFT 是 FWT 的一个小子集

参考：
[FMT 和 FWT 入门 - 洛谷专栏](https://www.luogu.com.cn/article/kh6xtkaj)
[快速沃尔什变换 - OI Wiki](https://oi-wiki.org/math/poly/fwt/)
[分圆多项式的简单性质与应用 - 知乎](https://zhuanlan.zhihu.com/p/105400582)

## FMT

参考：
[FMT（快速莫比乌斯变换） - 知乎](https://zhuanlan.zhihu.com/p/672965753#:~:text=FMT%20%E5%AE%9E%E8%B4%A8%E4%B8%8A%E6%98%AF%E4%B8%80%E7%A7%8D%E5%AE%B9%E6%96%A5%EF%BC%8C%E4%BB%8E%E6%96%87%E7%AB%A0%E5%BC%80%E5%A4%B4%E7%9A%84%E9%82%A3%E4%B8%80%E4%B8%AA%E7%BB%93%E8%AE%BA%E5%B0%B1%E6%98%AF%E5%AE%B9%E6%96%A5%E5%8E%9F%E7%90%86%E7%9A%84%E7%BB%8F%E5%85%B8%E7%BB%93%E8%AE%BA%E3%80%82%E6%9C%AC%E8%B4%A8%E4%B8%8A%EF%BC%8C%E5%AE%B9%E6%96%A5%E5%B0%B1%E6%98%AF%E5%B9%BF%E4%B9%89%E8%8E%AB%E6%AF%94%E4%B9%8C%E6%96%AF%E5%8F%98%E6%8D%A2%E7%9A%84%E4%B8%80%E7%A7%8D%E7%89%B9%E6%AE%8A%E5%BD%A2%E5%BC%8F%E3%80%82)
[SOSDP（子集DP）、高维前缀和、FMT（快速莫比乌斯变换）、FWT（快速沃尔什变换）学习笔记 - Troverld - 博客园](https://www.cnblogs.com/Troverld/p/14601821.html)