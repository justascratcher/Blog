---
tags:
  - 分裂搜索
---

[E - Cookies](https://atcoder.jp/contests/abc440/tasks/abc440_e)

第一眼：这是分裂搜索。但好像不太对。
再看一眼：转化一下就是。
解法：这就相当于是从一个长 $n\times10^{100}$ 的序列里取数，先考虑对于普通的 $|A|=n$ 的序列的 kth.

这里有一个经典的做法，就是先把最大的情况给出来，然后通过对这个情况微调得出次大的情况，并且同时保证所有可以微调出来的结果恰好能覆盖所有情况。基于这个方法直接用pq维护就行了。本质上就是在一个小根堆里找kth，只不过这里的堆是动态生成的。

对于这里，经典的做法是考虑把一个状态分成：
1. 前面一段致密的都选的前缀
2. 中间一个移动的选择
3. 后面的固定的选择
那么分裂时就只有固定2. 把1.的尾巴作为2.向后移动一格，和把2.向后移动一格两种可能，用上述方法维护就是 $O(k\log n)$ 的。

回到这里，向后移动一格就相当于直接跳到下一段，然后就做完了。

资料：  
[[An Optimal Algorithm for Selection in a Min-Heap.pdf]]  
[Fracturing Search - 洛谷专栏](https://www.luogu.me/article/zmwgsltb)  
[Fracturing Search - yozen & rah wiki](https://yozen0405.github.io/wiki/search/fracturing_search/)  

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
#define endl '\n'
const int mod=998244353, N = 3e5+5;
int n,a[N],k;
struct S {
    int sum,l,i,r;
    bool operator<(const S& v)const{
        return sum<v.sum;
    }
};
priority_queue<S> q;
signed main() {
    File(test);
    int x;
    cin>>n>>k>>x;
    For(i,1,n) cin>>a[i];
    sort(a+1,a+1+n,greater<int>());
    q.push({a[1]*k,k-1,1,n});
    while(x--){
        auto cur=q.top();q.pop();
        cout<<cur.sum<<endl;
        if (cur.l!=0&&cur.i>1){
            int sum=cur.sum-a[1]+a[2];
            q.push({sum,cur.l-1,2,cur.i});
        }
        if (cur.i+1<=cur.r){
            int sum=cur.sum-a[cur.i]+a[cur.i+1];
            q.push({sum,cur.l,cur.i+1,cur.r});
        }
    }
    return 0;
}
```