---
tags:
  - 思维/贪心
  - Hard/思路难想
  - multiset
---

[F - Egoism](https://atcoder.jp/contests/abc440/tasks/abc440_f)

> [!我错误的思路]-
>先考虑不带修改怎么做。
 显然一定能让 $\min\{n-1,\#\{b_i=2\}\}$ 匹马多一份贡献。
>
> >[!IDEA]
> >这里的 b = 1 有一种分割感，就是每过一个 b = 1 状态就好像被重置了。
>
考虑先把所有 b = 1 的马加入，顺序不重要，但这里为了方便从大到小。
然后把 b = 2 的马从大到小加入，每次都加在当前序列前面没有 b = 2 的最大 a 的马的前面，这样应该是最大的做法。
>
或者是乱序插入，但
如果 b = 2 那么如果自己比没有额外贡献的最大点还要大就把自己放到随便一个 b = 2 的前面，否则就放在没有额外贡献的最大值前面，
如果 b = 1 那么如果自己比有额外贡献的最小点还要小，那就放到尾巴上，否则把这个点插在有额外贡献的最小点前面。~~其实到这就错了~~
>
> >[!Proof]-
> >假设这样子给出的构造不是最优的，那么一定可以将这个序列的某一个多一份贡献的马的贡献给另一个，那么在构造的时候就应该就会把某一匹 b = 2 的马给到那只的前面，或者是把某只 b = 1 的马丢到尾巴上，与构造是矛盾的，故构造是最优的。
>
为了方便起见，先插入一个 a = 0, b = 1 的点。
>
> >[!Reflection]
> >这里有种局部性，因此可以贪心的考虑局部最优解。
>
这个方法也可以理解为
对于 b = 2 取没有额外贡献的最大的那个数替换成当前的。那么就可以用 BIT 维护，同时把最大数放到有贡献的 BIT 里，
对于 b = 1 的点，如果有比自己小的有额外贡献的最小的数，那就把那个数替换成自己并为他在无贡献 BIT 里新开一个点，否则自己在无贡献 BIT 里新开一个点。
>
接着如果带修改，只要能求出 没有额外贡献的最大数 和 有额外贡献的最小数 就可以了。
线段树二分求出即可，复杂度 $O(n\log n\log A)$ .
>（另外就是错在误以为修改是独立的）

正确思路：首先原题价值可以理解为是所有马的 a 之和，加上所有 b = 2 的马的后面的马的 a 之和，那么就是要优先把 a 大的数分配到 b = 2 的数的后面，令 $k=\#\{b_i=2\}$ 那么
1. 如果前 k 大的数都是 b = 2 的，那么其中最小的那个只能被放在最前被抛弃，取而代之的是 b = 1 里最大的那个（当然 k = n 时只抛弃最小的，没有再加上的数）。
2. 否则总是可以构造出让前 k 大的数都有额外贡献的方案。

那么问题就只剩下求前 k 大的数和，求第 k 大和第 k + 1 大的数，以及判断是否前 k 大的数都是 b = 2 的。

用两个 multiset 维护前 k 大的数和非前 k 大的数，再用**两个 multiset 维护所有 b = 2 和 b = 1 的数**，然后判断前 k 大的数是否都是 b = 2 的只要看 **b = 2 的 multiset 里最小的数是不是大于 b = 1 的 multiset 里最大的数**即可。

```cpp
const int mod=998244353, N = 3e5+5;
int n,q,a[N],b[N],sum=0,gksum=0;
multiset<int> kth[2],be[2];
void maintain(){
    int dif=be[1].size()-kth[1].size();
    For(i,1,dif){
        if (kth[0].empty())break;
        auto it=prev(kth[0].end());
        kth[1].insert(*it);
        gksum+=*it;
        kth[0].erase(it);
    }
    For(i,1,-dif){
        if (kth[1].empty())break;
        auto it=kth[1].begin();
        kth[0].insert(*it);
        gksum-=*it;
        kth[1].erase(it);
    }
}
void erase(int i) {
    sum-=a[i];
    be[b[i]-1].erase(be[b[i]-1].find(a[i]));
    auto it = kth[1].find(a[i]);
    if (it!=kth[1].end()){
        kth[1].erase(it);
        gksum-=a[i];
        return;
    }
    it = kth[0].find(a[i]);
    kth[0].erase(it);
}
void modify(int i,int A,int B){
    erase(i);
    sum+=A;
    a[i]=A,b[i]=B;
    be[b[i]-1].insert(A);
    if (kth[0].empty() || *kth[0].rbegin() < A)
        kth[1].insert(A),gksum+=A;
    else kth[0].insert(A);
    maintain();
}
signed main() {
    File(test);
    cin>>n>>q;
    For(i,1,n)cin>>a[i]>>b[i],sum+=a[i];
    For(i,1,n){
        be[b[i]-1].insert(a[i]);
        kth[0].insert(a[i]);
    }
    maintain();
    while (q--) {
        int w,x,y;cin>>w>>x>>y;
        modify(w,x,y);
        auto [mx,mn]=[&](){
            int mx=0;
            if (!kth[0].empty())mx=*prev(kth[0].end());
            int mn=0;
            if (!kth[1].empty())mn=*kth[1].begin();
            return make_pair(mx,mn);
        }();
        if (be[1].size()==n){
            cout<<sum+gksum-mn<<endl;
        }else if ([&](){
            if (be[1].empty()) return false;
            return *be[1].begin() >= *prev(be[0].end());
        }()) {
            cout<<sum+gksum-mn+mx<<endl;
        }else {
            cout<<sum+gksum<<endl;
        }
    }
    return 0;
}
```

相关代码技巧：
[[代码技巧]]