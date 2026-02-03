---
tags:
  - 笛卡尔树
  - 思维/观察性质
---

[E - Drop Min](https://atcoder.jp/contests/arc212/tasks/arc212_e)

思路：就是把笛卡尔树建出来，两个子树显然是独立的，然后每个节点显然要等到某个子树清空之后才可能弹出，所以一个子树的弹出方案数就是
1. 只有左边有大于自己的值时，这个节点相当于是附在了左子树序列尾巴上，方案数是 $ans_lans_r{siz_l+siz_r+1\choose siz_r}$ 
2. 只有右边有时，同理方案数是 $ans_lans_r{siz_l+siz_r+1\choose siz_l}$ 
3. 左右都有时，只要左右有一个弹完了这个节点就可以弹出。容斥一下就是从左边弹的方案数+从右边弹的方案数-左右都被弹完才弹出这个节点的方案数，也就是方案数是 $ans_lans_r({siz_l+siz_r+1\choose siz_r}+{siz_l+siz_r+1\choose siz_l}-{siz_l+siz_r\choose siz_r})$ 
4. 最后如果两边都没有，系数就是混合左右子树，方案数就是 $ans_lans_r{siz_l+siz_r\choose siz_r}$ 
然后算就完了。

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
const int N = 3e5,mod = 998244353;
int fac[N],inv[N];
int n,p[N],L[N],R[N],siz[N];
int stk[N],ns;
int qpow(int a,int b){
    if (b==0)return 1;
    int v=qpow(a,b>>1);
    return (ll)v*v%mod*((b&1)?a:1)%mod;
}
void init(int n) {
    fac[0]=1;
    For(i,1,n)fac[i]=(fac[i-1]*i)%mod;
    inv[n]=qpow(fac[n],mod-2);
    rFor(i,0,n-1)inv[i]=(inv[i+1]*(i+1))%mod;
}
int C(int n,int m){
    return fac[n]*inv[n-m]%mod*inv[m]%mod;
}
void dfs1(int u){
    if(L[u])dfs1(L[u]);
    if(R[u])dfs1(R[u]);
    siz[u]=siz[L[u]]+siz[R[u]]+1;
}
int dfs(int u,bool l,bool r){
    int al=1,ar=1;
    if (L[u])al=dfs(L[u],l,1);
    if (R[u])ar=dfs(R[u],1,r);
    int f=0,sl=siz[L[u]],sr=siz[R[u]];
    if(l)f=(f+C(sl+sr+1,sr))%mod;
    if(r)f=(f+C(sl+sr+1,sl))%mod;
    if(l&&r)f=(f-C(sl+sr,sr))%mod;
    if((!l) && (!r))f=C(sl+sr,sl); // 关键，不是 1
    return f*al%mod*ar%mod;
}
signed main() {
    File(test);
    cin >> n;
    init(N-1);
    For(i,1,n)cin>>p[i];
    stk[++ns]=1;
    For(i,2,n){
        bool hv=0;
        while (ns&&p[stk[ns]]<p[i])ns--,hv=1;
        if (ns)R[stk[ns]]=i;
        if (hv)L[i]=stk[ns+1];
        stk[++ns]=i;
    }
    dfs1(stk[1]);
    cout<<(dfs(stk[1],0,0)+mod)%mod<<endl;
    return 0;
}
```