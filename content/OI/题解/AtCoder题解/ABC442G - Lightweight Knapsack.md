---
tags:
  - 三分
  - 背包问题
  - 思维/贪心
  - 打包背包
  - Hard/难写
  - need_another_impl
---

[G - Lightweight Knapsack](https://atcoder.jp/contests/abc442/tasks/abc442_g)

思路 1：因为 w 只有三种，而同种 w 肯定优先取 v 大的，于是把物品按照 w 分好类，排好序
求出前 k 大的物品的和可以预处理后 O(log n) 得到
如果总 w < c 直接输出 $\sum v$
否则假设取 a 个 w=1 的物品，那么剩下的空间给 w=2 和 w=3 的填，发现在 c 固定的情况下， $\sum v$ 关于 w=2 的物品数是凸的，可以三分得到
然后得到的结果再与 w=1 物品的价值求和后关于 a 又是凸的，于是再次三分，复杂度 $O(n\log^3n)$ ，死了
(貌似可以把上次的w=2和w=3的分界记录下来，把复杂度降到 $O(n\log^2n)$ )

思路 2：其实是一开始有的一个想法，就是把每种物品都打包成 w = 6 的物品，但是想想会有剩余的数就没想下去，结果看了题解就是这样做。
只要考虑三种物品打包到 w = 6 后的余下的未打包的物品数量即可，记作 k ，那么可以想成前 k 个必选，后面每 w = 6 就组一组，那么就是在一堆 w = 6 里选，直接排好序从大到小取即可
这里不同的 k 的组合只有 6 \* 3 \* 2 = 36 种，直接暴力搜索即可，复杂度 O(n log n)

Bonus：[[打包背包]]

这个调了我好久
```cpp
const int N=2e5+5;
pii item[4][N];
int nit[4]{},s[4];
int n,c;
auto pack(pii item[], int n, int r, int grp) {
    vector<pii> ret;
    int rsti=0,rsts=0,sum=0;
    For(i,1,n){
        int k=item[i].r-r;
        if (k<=0){
            sum+=item[i].l*item[i].r;
            r=-k;
            continue;
        }else if (r!=0){
            sum+=item[i].l*r;
            r=0;
        }
        if (k+rsti<grp){
            rsti+=k;
            rsts+=item[i].l*k;
        } else {
            int ng=(k+rsti)/grp;
            ret.pb({rsts+item[i].l*(grp-rsti),1});
            if(ng>1)ret.pb({item[i].l*grp,ng-1});
            rsti=(k+rsti)%grp;
            rsts=rsti*item[i].l;
        }
    }
    if(r)sum+=item[n].first*r;
    return make_pair(ret,sum);
}
signed main() {
    File(test);
    cin>>n>>c;
    For(i,1,n){
        int w,v,k;cin>>w>>v>>k;
        item[w][++nit[w]]={v,k};
        s[w]+=k;
    }
    For(i,1,3)sort(item[i]+1,item[i]+nit[i]+1,greater<pii>());
    int tans=0;
    For(i,0,min(5ll,s[1]))For(j,0,min(2ll,s[2]))For(k,0,min(1ll,s[3])){
        int siz=c-i-j*2-k*3;
        if (siz<0)continue;
        auto [v1,sum1]=pack(item[1],nit[1],i,6);
        auto [v2,sum2]=pack(item[2],nit[2],j,3);
        auto [v3,sum3]=pack(item[3],nit[3],k,2);
        int ans=sum1+sum2+sum3;
        vector<pii> res1(v1.size()+v2.size());
        vector<pii> res(v1.size()+v2.size()+v3.size());
        merge(rg(v1),rg(v2),res1.begin(),greater<pii>());
        merge(rg(res1),rg(v3),res.begin(),greater<pii>());
        int l=0;
        while(siz>0){
            if (l>=res.size())break;
            auto [v,k]=res[l];
            if (siz>=k*6)siz-=k*6,ans+=v*k;
            else {
                ans+=v*(siz/6);
                break;
            }
            l++;
        }
        tans=max(tans,ans);
    }
    cout<<tans<<endl;
}
```

三分写法？：
```cpp

```