---
tags:
  - 图论/拓扑图
  - Hard/大码量
  - Hard/思路混乱
  - 动态规划_DP
  - 分类讨论
---

[G - Haunted House](https://atcoder.jp/contests/abc440/tasks/abc440_g)

思路：图建出来，缩点后就是个拓扑图。

Idea 1：
考虑把每个 u 复制一个点 u'，然后在有 (u, v) 的边就连一条 (v, u') 和 (u', v') 的边，那么就变成了分层图，这里权值最多多算一个 v，但是只要在搜素时记录一下用梯子的点，减掉即可。最后答案就是从某点出发的最长路。
复杂度 $O((NMF)^2)$ 还不行。

Idea 2：
再考虑一下，不妨是理解为拓扑图 dp 一样的东西，每个点都 dp 出不用梯子的最大答案和次大答案，并记录对应的第一步走的节点。
接着 dp 出用梯子的方案。
首先考虑当前点用梯子的方案，那么对于反向边联通的相邻节点，如果它的最优方案是经过当前点的，那答案就是 $\max\{相邻节点的最优方案,自己的价值+相邻节点的次优方案\}$
否则就是 $相邻节点的最优方案+自己的价值$ 。
其次，如果是当前节点的父节点（这里包含父节点的前继）用梯子，那么只要看最优方案是否是向自己搭梯子，
1. 如果是，
	1. 如果自己不搭梯子的最优方案经过父节点，答案就是 $\max\{自己的最优方案,父节点的价值+自己的次优方案\}$
	2. 否则，答案就是 $父节点的价值+自己的最优方案$ 
	但上面两种方案都要与 $父节点的搭梯子次优方案+自己的价值$ 再取 max
2. 如果不是，那答案就是 $自己的价值+父节点的最优方案$ 

最后，答案就是两种取 max，询问时查下点的缩点的编号对应的 dp 就行了。

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
const int mod=998244353, N = 505, F = 15;
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
namespace G{
    int E[N*N*F*2],ne[N*N*F*2],h[N*N*F],rh[N*N*F],NNN;
    void add(int u,int v){
        E[NNN]=v,ne[NNN]=h[u],h[u]=NNN++;
        swap(u,v);
        E[NNN]=v,ne[NNN]=rh[u],rh[u]=NNN++;
    }
}

int f,h,w;
char mp[F][N][N];
int color[F][N][N];
bool vis[F][N][N];
int sum[F*N*N];
int mx[F*N*N][2];
int dp[F*N*N][2][2],p[F*N*N][2][2];// [][0] ladder not used, [][1] ladder used
int ans[F*N*N];
unordered_set<int> fnd;
void dfs(int z,int x,int y,int col){
    if (vis[z][x][y]) return;
    color[z][x][y]=col;
    vis[z][x][y] = 1;
    sum[col]+=mp[z][x][y]-'0';
    if (z < f) {
        if (mp[z+1][x][y]!='#')fnd.insert(color[z+1][x][y]);
    }

    int dx[]={-1,1,0,0};
    int dy[]={0,0,-1,1};
    For(dir,0,3){
        int nx=x+dx[dir],ny=y+dy[dir];
        if(nx<=0||nx>h||ny<=0||ny>w)continue;
        if(mp[z][nx][ny]=='#') continue;
        if(vis[z][nx][ny])continue;
        dfs(z,nx,ny,col);
    }
}
void ins(int u, int t,int val,int v) {
    if (dp[u][t][0] <= val) {
        dp[u][t][1] = dp[u][t][0];
        p[u][t][1] = p[u][t][0];
        dp[u][t][0] = val;
        p[u][t][0] = v;
    } else if (dp[u][t][1] <= val) {
        dp[u][t][1] = val;
        p[u][t][1] = v;
    }
}
int d(int u,int t,int i){
    return dp[u][t][i];
}
int P(int u,int t,int i){
    return p[u][t][i];
}
signed main() {
    ms(G::h,-1);
    ms(G::rh,-1);
    File(test);
    cin >> f >> h >> w;
    For(i,1,f){
        For(j,1,h){
            For(k,1,w){
                cin>>mp[i][j][k];
            }
        }
    }
    int col=1;
    rFor(i,1,f){
        For(j,1,h){
            For(k,1,w){
                if (!vis[i][j][k]&&mp[i][j][k] != '#') {
                    dfs(i,j,k,col);
                    aFor(v,fnd)G::add(col,v);
                    unordered_set<int> nw{};
                    swap(fnd,nw); // a trick
                    col++;
                }
            }
        }
    }

    For(u,1,col) {
        ins(u,0,sum[u],0);
        eFor(i,u){
            int v=G::E[i];
            ins(u,0,d(v,0,0)+sum[u],v);
        }
    }

    For(u,1,col) {
        ins(u,1,sum[u],0);
        erFor(i,u){
            int v=G::E[i];
            if (P(v,0,0)==u) ins(u,1,max(d(v,0,0),d(v,0,1)+sum[u]),v);
            else ins(u,1,d(v,0,0)+sum[u],v);
        }
        eFor(i,u){
            int v=G::E[i];
            if (P(v,1,0)==u){
                int mx = 0;
                if (P(u,0,0)==v)mx=max(d(u,0,0),d(u,0,1)+sum[v]);
                else mx=d(u,0,0)+sum[v];
                mx=max(mx,d(v,1,1)+sum[u]);
                ins(u,1,mx,v);
            } else ins(u,1,sum[u]+d(v,1,0),v);
        }
        ans[u] = max(d(u,0,0),d(u,1,0));
    }
    
    int q;cin>>q;
    while(q--){
        int g,a,b;cin>>g>>a>>b;
        cout<<ans[color[g][a][b]]<<endl;
    }
    return 0;
}
```

相关代码技巧：
[[代码技巧]]