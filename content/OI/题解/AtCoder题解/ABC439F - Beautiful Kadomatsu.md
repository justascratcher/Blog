---
tags:
  - 思维/转化
---

[F - Beautiful Kadomatsu](https://atcoder.jp/contests/abc439/tasks/abc439_f)

题意其实就是要求一个子序列开头和结尾一个上升，一个下降，中间没有要求，故直接统计即可。（有点诈骗）

```c++
#include <bits/stdc++.h>
using namespace std;
#define int long long
#define ll long long
#define endl '\n'
#define IOopt ios::sync_with_stdio(false),cin.tie(0),cout.tie(0)
#define File(name)freopen(#name".in","r",stdin),freopen(#name".out","w",stdout),IOopt
#define ms(v,u)memset(v,u,sizeof v)
#define For(i,a,b)for(int i=a;i<=b;i++)
#define rFor(i,a,b)for(int i=b;i>=a;i--)
#define aFor(i,c)for(auto&i:c)
#define rg(v) v.begin(), v.end()
#define eFor(i,u,g)for(int i=g.h[u];~i;i=g.ne[i])
#define popcnt(u)__builtin_popcount(u)
#define ctz(u)__builtin_ctz(u)
#define l first
#define r second
const int N=3e6+5;
int v[N];
int n;
int bit[N];
int lowbit(int u){return u&(-u);}
int qry(int u) {
    int ans = 0;
    for (;u>0;u-=lowbit(u)){
        ans = ans + bit[u];
    }
    return ans;
}
void mdf(int u, int v){
    for(;u<N;u+=lowbit(u)){
        bit[u] = bit[u] + v;
    }
}
int pr[N],ne[N],ppre[N];
void solve(){
    cin >> n;
    For(i,1,n) {
        cin >> v[i];
    }
    int ans = 0;
    For(i,1,n){
        pr[i]=qry(v[i]-1);
        mdf(v[i],1);
    }
    For(i,1,n){
        mdf(v[i],-1);
    }
    rFor(i,1,n){
        ne[i]=qry(v[i]-1);
        mdf(v[i],1);
    }
    For(i,1,n){
        ppre[i]=ppre[i-1]*2+pr[i];
        ppre[i]%=998244353;
        ans=(ans+(ppre[i-1]+pr[i])*ne[i])%998244353;
    }
    cout << ans << endl;
}
signed main(){
    File(test);
    solve();
    return 0;
}
```

