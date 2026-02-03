
```cpp
#include <bits/stdc++.h>
using namespace std;

#define IOopt ios::sync_with_stdio(false),cin.tie(0),cout.tie(0)
#define File(name) freopen(#name".in","r",stdin),freopen(#name".out","w",stdout),IOopt
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
#define i128 __int128
#define vi vector<int>
#define Vec vector
#define pii pair<int,int>
#define endl '\n'
const int mod=998244353, N = 1e6; 
int qpow(int a,int b){
    if (b==0)return 1;
    int v=qpow(a,b>>1);
    return (ll)v*v%mod*((b&1)?a:1)%mod;
}

void chkmx(int&a,int b){a=max(a,b);}
void chkmn(int&a,int b){a=min(a,b);}

struct dbger{
	dbger&operator<<(int v){cerr<<v<<" ";return*this;}
	dbger&operator<<(char v){cerr<<v<<" ";return*this;}
    dbger&operator<<(const string& v){cerr<<v<<endl;return*this;}
    template<typename T1,typename T2>
    dbger&operator<<(const pair<T1,T2>& v){cerr<<"pair("<<v.l<<","<<v.r<<")";return*this;}
	template<typename T>
	dbger&operator<<(const vector<T>& v){aFor(i,v)(*this)<<i;cerr<<endl;return*this;}
}dbg; 

signed main() {
	File(test);
	
	return 0;
}
```

