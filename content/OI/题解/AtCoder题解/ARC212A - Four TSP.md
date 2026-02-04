---
tags:
  - 简单
  - 思维/观察性质
---

[A - Four TSP](https://atcoder.jp/contests/arc212/tasks/arc212_a)

这题把我坑死了

思路：发现环只有三种，并且不相邻的边必须同时被走过或者不被走过，所以可以一对一对枚举，一共就 $O(k^2)$ 个三元组，最后，一组的价值就是三个里面最小的两个的和，系数就是 $(t-1)(s-1)(p-1)$ （t, s, p 是三对边的权值和）

```c++
#include <bits/stdc++.h>
using namespace std;

#define File(name) freopen(#name".in","r",stdin),freopen(#name".out","w",stdout),ios::sync_with_stdio(false),cin.tie(0),cout.tie(0)
#define For(i,a,b)for(int i = a; i <= b; i++)
#define rFor(i,a,b)for(int i=b;i>=a;i--)
#define aFor(i,v) for(auto&i:v)
#define eFor(i,u) for(int i=h[u];~i;i=ne[i])
#define l first
#define r second
#define ll long long
#define popcnt __builtin_popcount
#define ctz __builtin_ctz
#define rg(v) v.begin(),v.end()
#define int long long
const int N = 1e5,mod = 998244353;
signed main() {
    int k, ans=0; cin >> k;cerr<<k;
    For(p,2,k) {
        For(s, 2, k) {
            if (p+s > k-2) break;
            ans = (ans + (min(min(p+s,k-s),k-p)) * (p - 1) % mod * (s - 1) % mod * (k-p-s-1)%mod) % mod;
        }
    }
    cout << ans%mod<<endl;
    return 0;
}
```

