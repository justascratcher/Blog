---
tags:
  - 思维/转化
  - 图论/最短路
---

[B - Stones on Grid](https://atcoder.jp/contests/arc212/tasks/arc212_b)

思路：原题意很恶心，但是注意到要求第 i 行和第 i 列的石子数相同就是说，放一个石子的代价等价于是在一个序列的 $a_i$ 处 +1 在 $b_i$ 处 -1，最后序列的所有的点都是 0. 那么有点像是流动，连 $a_i$ 向 $b_i$ 的边后，每个点他的出度和入度相同. 那么最小的结果一定是一个环，而环的一边已经给出，剩下的就用 Dijkstra 跑一遍最短路就搞定了。

```cpp
const int N = 2e5+5,mod = 998244353;
int n,m;
int E[N],ne[N],W[N],h[N],NNN;
void add(int u, int v, int w){
    E[NNN]=v,W[NNN]=w,ne[NNN]=h[u],h[u]=NNN++;
}
bool vis[N];
int d[N];
signed main() {
    File(test);
    ms(h,-1);
    ms(d,0x3f);
    cin>>n>>m;
    int s,t,ans;cin>>t>>s>>ans;
    For(i,2,m){
        int x,y,c;cin>>x>>y>>c;
        add(x,y,c);
    }
    struct Node {
        int len, i;
        bool operator > (const Node& v)const {
            return len > v.len;
        }
    };
    priority_queue<Node,vector<Node>, greater<Node>> q;
    q.push({0,s});
    d[s]=0;
    while (!q.empty()) {
        auto cur=q.top();q.pop();
        if (vis[cur.i])continue;
        vis[cur.i]=1;
        eFor(i,cur.i) {
            int v=E[i],w=W[i];
            if (vis[v])continue;
            if (d[v] <= cur.len + w) continue;
            d[v] = cur.len + w;
            q.push({d[v],v});
        }
    }
    if (d[t] > 1e16) {
        cout << -1;
    } else {
        cout << d[t] + ans << endl;
    }
    return 0;
}
```

