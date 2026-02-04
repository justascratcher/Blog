---
tags:
  - 思维/不同方向思考
  - 简单
---

[D - Forbidden List 2](https://atcoder.jp/contests/abc440/tasks/abc440_d)

直接二分。但是在值域上算是 $O(\log^2n)$ 的会被卡，换成对序列中的点二分之后再补上剩余的数是 $O(\log n)$ 的可以通过。

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
int n,a[N];
int cnt(int v){
    return upper_bound(a+1,a+n+1,v)-a-1;
}
signed main() {
    int q;
    cin >> n >> q;
    For(i,1,n) cin>>a[i];
    sort(a+1,a+1+n);
    while(q--){
        int x,y;cin>>x>>y;
        int c=cnt(x-1);
        int l=c,r=n;
        while(l<r){
            int mid=(l+r+1)>>1;
            if(a[mid]-x+1-(mid-c)>=y){
                r=mid-1;
            } else {
                l=mid;
            }
        }
        cout<<x+y-1+(l-c)<<endl;
    }
    return 0;
}
```

TLE的：
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
int n,a[N];
int cnt(int v){
    return upper_bound(a+1,a+n+1,v)-a-1;
}
signed main() {
    int q;
    cin >> n >> q;
    For(i,1,n) cin>>a[i];
    sort(a+1,a+1+n);
    while(q--){
        int x,y;cin>>x>>y;
        int c=cnt(x-1);
        int l=x,r=x+n+y,ans=0;
        while(l<r){
            int mid=(l+r)>>1,v=mid-x+1-cnt(mid)+c;
            cerr<<x<<" "<<mid<<" "<<v<<endl;
            if (v<y){
                l=mid+1;
            }else if (v>y){
                r=mid;
            } else {
                r=mid;
                ans=mid;
            }
        }
        cout << ans << endl;
    }
    return 0;
}
```