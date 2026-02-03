---
tags:
  - 思维/乱搞
---

[D - Two Rooms](https://atcoder.jp/contests/arc212/tasks/arc212_d)

没啥办法，直接瞎搞就行。
就是如果一个人不满足要求就直接放到对面房间就行。
具体证明就是这个操作会把总好感度增加，而总好感度是 $O(n^2A)$ 的(不是 $O(nA)$ )，故不会超时。

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
const int N = 60,mod = 998244353;
int n,v[N][N];
bool lr[N];
signed main() {
    File(test);
    cin >> n;
    For(i,1,n)For(j,1,n)cin>>v[i][j];
    while (1) {
        bool flg=1;
        For(i,1,n){
            int s=0;
            For(j,1,n){
                s+=lr[i]==lr[j]?v[i][j]:-v[i][j];
            }
            if (s<0)lr[i]=!lr[i],flg=0;
        }
        if (flg)break;
    }
    For(i,1,n){
        cout<<(lr[i]?'X':'Y');
    }
    return 0;
}
```