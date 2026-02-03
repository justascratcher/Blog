---
tags:
  - 思维/基础拓展
  - 背包问题
---

[F - Must Buy](https://atcoder.jp/contests/abc441/tasks/abc441_f)

思路：在普通背包基础上记录转移的可能来向，然后据此判断是一定要，可能要，还是一定不要。

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
#define pb push_back
#define popcnt __builtin_popcount
#define ctz __builtin_ctz
#define rg(v) v.begin(),v.end()
#define int long long
#define pii pair<int,int>
#define endl '\n'
const int mod=998244353, N = 1000+5,M=5e4+5;
int lowbit(int u) {return u&(-u);}
int qpow(int a,int b){
    if (b==0)return 1;
    int v=qpow(a,b>>1);
    return (ll)v*v%mod*((b&1)?a:1)%mod;
}
int f[M][N];
bool dwn[M][N];
bool lef[M][N];
bool vis[M][N];
int type[N];
int p[N],v[N],n,m;
void dfs(int x,int i){
    if (i==0)return;
    if (vis[x][i])return;
    vis[x][i]=1;
    int d=dwn[x][i],l=lef[x][i];
    switch (type[i]){
    case 0:
        if (l&&d)type[i]=2;
        if(l&&!d)type[i]=1;
        if(!l&&d)type[i]=3;
        break;
    case 1:
        if(d)type[i]=2;
        break;
    case 3:
        if(l)type[i]=2;
        break;
    }
    if (d)dfs(x-p[i],i-1);
    if (l)dfs(x,i-1);
}
signed main() {
    cin>>n>>m;
    For(i,1,n)cin>>p[i]>>v[i];
    int mans=0;
    For(x,0,m){
        For(i,1,n){
            if (x<p[i]){
                f[x][i]=f[x][i-1];
                dwn[x][i]=0;
                lef[x][i]=1;
            }else {
                f[x][i]=max(f[x][i-1],v[i]+f[x-p[i]][i-1]);
                if (f[x][i]==v[i]+f[x-p[i]][i-1]){
                    dwn[x][i]=1;
                }
                if(f[x][i]==f[x][i-1]){
                    lef[x][i]=1;
                }
            }
            mans=max(mans,f[x][i]);
        }
    }
    For(x,0,m){
        For(i,1,n){
            if (f[x][i]==mans){
                dfs(x,i);
            }
        }
    }
    For(i,1,n){
        if (type[i]==0){
            type[i]=1;
        }
        cout<<char((4-type[i])+'A'-1);
    }
    return 0;
}
```