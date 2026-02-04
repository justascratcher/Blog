---
tags:
  - 生成函数_GF
  - unsolved
  - 快速数论变换_NTT
  - 多项式/多项式求逆
  - 多项式/Bostan-Mori
  - 二维NTT
  - Hard
---

[G - Sugoroku 6](https://atcoder.jp/contests/abc439/tasks/abc439_g)

# Idea 1:

拆成 $\sum_{k\in [1,n]}(k次不成)^{i-1}(k次成)(k-1次不成)^{n-i}$ ，然后计算小于等于 k 次成功的概率差分算出 $a_k$ 为恰在 k 次成功的的概率。
然后答案就是 $\sum \limits_{k=1}^n(1-a_k)^{i-1}a_k(1-a_{k-1})^{n-i}$ ，接着化简得 $\sum \limits_{k=1}^n\left(\dfrac{1-a_k}{1-a_{k-1}}\right)^{i}\dfrac{a_k}{1-a_k}(1-a_{k-1})^n$
但后面不会做。(此时的我已经混淆了 N 和 L)
# 看了题解：

> 然后计算小于等于 k 次成功的概率差分算出 $a_k$ 为恰在 k 次成功的的概率。  

这句话是对的，不过是算失败的概率。  

>[!IDEA]  为什么是失败的概率
>其实就是因为生成函数更适合解决小于的问题而不是大于的问题

具体地，令 $b_k$ 是在玩 k 次后仍未达到 N 的概率，有  
$$
\begin{aligned}
b_k&=\sum\limits_{i=0}^{N-1}[x^i](\frac1m\sum_jx^{A_j})^k\\
&=[x^{N-1}]\frac1{1-x}(\frac1m\sum_jx^{A_j})^k\\
\end{aligned}
$$
然后考虑 b 的生成函数 $F(y)$  
$$
\begin{aligned}
F(y)&=\sum b_iy^i\\
&=\sum[x^{N-1}]\frac1{1-x}(\frac1m\sum_jx^{A_j})^iy^i&(令H(x)=\frac1m\sum_jx^{A_j})\\
&=[x^{N-1}]\frac1{1-x}\sum (yH(x))^i\\
&=[x^{N-1}]\frac1{(1-x)(1-yH(x))}\\
\end{aligned}
$$
接下来只要有办法求出 F 的各项系数即可，想法是直接用 [[Bostan-Mori 分式远处系数 & 常系数齐次线性递推 & 多项式复合 & 多项式复合逆#基本思路|二元 Bostan-Mori]] 求，只要求到 $\lceil N/min(a)\rceil$ 即可  
后面是求答案  
答案就是   
$$\sum_{k\in [1,n]}(k次不成)^{i-1}(恰好k次成)(k-1次不成)^{L-i}=\sum_{k=1}^nb_k^{i-1}(b_k-b_{k-1})b_{k-1}^{L-i}=\sum_{k=1}^n\left(\frac{b_k}{b_{k-1}}\right)^{i-1}(b_k-b_{k-1})b_{k-1}^{L-1}$$ 
那么设 $p_k=\frac{b_k}{b_{k-1}}$，$q_k=(b_k-b_{k-1})b_{k-1}^{L-1}$，那么就是要对于 i = 0 ... n - 1 (这里的 n 指 k, 即题面的 L)求 $ans_i=\sum_{k=1}^np_k^{i}q_k$，这里题解说继续套 GF
令 K(x) 是 $ans_i$ 的 OGF，那么  
$$
\begin{aligned}
K(x)&=\sum_i ans_ix^i\\
&=\sum_i x^i\sum_{k=1}^np_k^{i}q_k\\
&=\sum_i \sum_{k=1}^n(xp_k)^{i}q_k\\
&=\sum_{k=1}^n\sum_i (xp_k)^{i}q_k\\
&=\sum_{k=1}^n\frac{q_k}{1-xp_k}\\
\end{aligned}
$$
接着 K(x) 其实是很多分式的和，如果把每个分式都直接打开算，复杂度是 $O(n^2)$ 的会死掉，那么可以把分式分治地（其实就是先把指数小的合成指数翻倍的，）每次两两合并成 $\frac{M(x)}{N(x)}$ 的形式，然后把 N 求逆再乘上 M 得到各项系数。  
  
这个是会 WA + TLE 的，以后回来再改了  
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
// #define int long long
#define pii pair<int,int>
#define endl '\n'
const int mod=998244353, N = 1<<19; // not 2e5, due to alignment to 2^n


int qpow(int a,int b){
    if (b==0)return 1;
    int v=qpow(a,b>>1);
    return (ll)v*v%mod*((int(b)&1)?a:1)%mod;
}

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
int cnt=0,tl,ttl;
template<typename T>
struct Poly {
    vector<T> c;
    int deg;
    explicit Poly(int n=0){
        resize(n);
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
        tl=lst,ttl=l;
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
        For(i,0,G.deg) G[i]=(i+1)*F[i+1];
        return G;
    }

    Poly I() {
        Poly& F=*this;
        Poly G(F.deg+1);G[0]=0;
        For(i,1,G.deg) G[i]=F[i-1]*inv[i];
        return G;
    }

    friend std::ostream& operator<<(std::ostream& os, const Poly& F) {
        For(i,0,F.deg) os<<F[i]<<"\n"; return os;
    }
    friend std::istream& operator>>(std::istream& os, Poly& F) {
        For(i,0,F.deg) os>>F[i]; return os;
    }
};
using poly = Poly<mint>;

poly inverse(poly F, int len) {
    poly G(0); G[0] = qpow(F[0],mod-2);
    for(int i=1;i<len;i<<=1)G=(2*G-G*G*F).resize(i);
    return (2*G-G*G*F).resize(len-1);
}

struct poly2 : public Poly<poly> {
    poly2(Poly<poly> v){
        c.resize(v.deg+1);
        For(i,0,v.deg)c[i]=v[i];
        deg=v.deg;
    }
    poly2(int n,int m=0) : Poly<poly>(n) {
        For(i,0,n)c[i].resize(m);
        deg=n;
    }
    poly2& NTT(int inv) { // inv = 1 NTT, inv = -1 INTT
        poly2&F=*this;
        transpose();
        For(i,0,F.deg) F[i].NTT(inv);
        transpose();
        For(i,0,F.deg) F[i].NTT(inv);
        return *this;
    }
    poly2& transpose() { 
        poly2&F=fill();
        vector<vector<mint>> mat(F.deg+1,vector<mint>(F[0].deg+1));
        For(i,0,F.deg){
            For(j,0,F[0].deg){
                mat[i][j]=F[i][j];
            }
        }
        F.resize(mat[0].size()-1);
        For(i,0,F.deg) {
            F[i].resize(mat.size()-1);
            For(j,0,mat.size()-1) F[i][j]=mat[j][i];
        }
        return *this;
    }
    poly2& fill() {
        poly2&F=*this;
        int mx=0;
        For(i,0,F.deg) mx=max(mx,F[i].deg);
        For(i,0,F.deg) F[i].resize(mx);
        return *this;
    }
    poly2& operator*=(poly2 G) {
        poly2& F=*this;
        int len=F.deg+G.deg;
        F.resize(len),G.resize(len);
        int mx=0;
        For(i,0,len) mx=max(mx,F[i].deg+G[i].deg);
        For(i,0,len) F[i].resize(mx),G[i].resize(mx);
        F.NTT(1),G.NTT(1);
        
        For(i,0,len)
            For(j,0,F[0].deg) 
                F[i][j]*=G[i][j];
        
        F.NTT(-1);
        return *this;
    }
};

poly DivAt(poly2 F, poly2 G, int k, int len) {
    while(k) {
        poly2 iG=G;
        for(int i=1;i<=iG.deg;i+=2)iG[i]=-iG[i];
        F*=iG,G*=iG;
        int i=k&1;
        k>>=1;
        for(;i<=F.deg;i+=2)F[i>>1]=F[i];
        F.resize(min((i-2)>>1,k));// min, to ensure the time complexity
        for(i=0;i<=G.deg;i+=2)G[i>>1]=G[i];
        G.resize(min((i-2)>>1,k));
    }
    return (F[0]*inverse(G[0],len)).resize(len-1);
}
int n,m,k;
int a[N];

struct Frac {
    poly p,q;
};

Frac merge(Frac a, Frac b){
    return Frac{a.p*b.q+a.q*b.p,a.q*b.q};
}
poly b;
Frac ans(int l,int r){
    if (l==r) {
        poly p(0),q(1);
        p[0]=(b[l-1]-b[l])*qpow(b[l-1],k-1);
        q[0]=1,q[1]=-b[l]*qpow(b[l-1],mod-2);
        return {p,q};
    }
    int mid=(l+r)>>1;
    return merge(ans(l,mid),ans(mid+1,r));
}

signed main() {
    init(); // init important
    File(test);
    cin>>n>>m>>k;
    
    // P=1-yH(x)
    poly2 P(n);
    For(i,0,n)P[i].resize(2);
    P[0][0]=1;
    int im=qpow(m,mod-2);
    int ma=1e9+7;
    For(i,1,m)cin>>a[i],ma=min(ma,a[i]),P[a[i]][1]-=im;
    // fac=1-x
    poly2 fac(2);
    fac[0].resize(1)[0]=1;
    fac[1].resize(2)[0]=-1;

    poly2 f(0);
    f[0].resize(0)[0]=1;

    // b=[x^n-1]1/((1-x)(1-yH(x)))
    b=DivAt(f,fac*P,n-1,n);

    Frac ams=ans(1,k);
    cout<<(inverse(ams.q,k)*ams.p).resize(k-1)<<endl;
    return 0;
}
```