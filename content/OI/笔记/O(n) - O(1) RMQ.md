---
tags:
  - RMQ
  - 最近公共祖先_LCA
---
很典的一个 trick
把区间 log n 为一块分块，然后对块与块之间做 ST 表，复杂度是 $O(\frac n{\log n}\log(\frac n{\log n}))=O(n)$
那么所有跨区间的 RMQ 就直接把中间的用 ST 表算出来，两边剩余的区间直接用前后缀最大值处理就行
这里**关键的问题其实是如果一次 RMQ 只在一个小区间内查怎么办**
首先直接做的话肯定是 O(log n)
这里奇妙的想法是把单调栈的结果存下来，那么这里想要求出 l - r 的 RMQ 其实就是把 r 结尾的单调栈里面大于 l 的第一个下标取出来，对应点的值就是答案
（这个其实就是滑动窗口的想法）
## 代码实现
```cpp
// 此处省略原题的 gen
const int mod=998244353, N = 2e7+5000, LOGN=25; 
int a[N], ST[N/LOGN][LOGN],pre[N],suf[N], q[N],stk[70],nstk;
int n,m,s,L;
int st(int l,int r){
    int k=lg(r-l+1);
    return max(ST[l][k],ST[r-(1<<k)+1][k]);
}
int qry(int l,int r){
    int bl=(l-1)/L,br=(r-1)/L; // 这里直接算出 id 比去查表更快
    if (bl==br){
        int i=L*bl;
        l-=i;
        return a[i+ctz((q[r]>>l)<<l)];
    } else {
        return max(max(suf[l],pre[r]),bl==br-1?0:st(bl+1,br-1));
    }
}
signed main() {
    File(test);
    cin>>n>>m>>s;
    srand(s);
    For(i,1,n)a[i]=read();
    L=log2(n)+1;
    for(int i=0;i<=n;i+=L){
        nstk=0;
        int sta=0;
        For(j,1,L) {
            while(nstk&&a[i+stk[nstk]]<=a[i+j])sta^=1<<stk[nstk], nstk--;
            stk[++nstk]=j;
            sta^=1<<j;
            q[i+j]=sta;
        }
        pre[i+1]=a[i+1];
        For(j,2,L) pre[i+j]=max(a[i+j],pre[i+j-1]);
        suf[i+L]=a[i+L];
        rFor(j,1,L-1) suf[i+j]=max(a[i+j],suf[i+j+1]);
        ST[i/L][0]=a[i+stk[1]];
    }
    int mx=log2(n/L)+1,nn=n/L+1;
    For(j,1,mx) For(i,1,nn-(1<<j)+1) ST[i][j]=max(ST[i][j-1],ST[i+(1<<j-1)][j-1]);

    unsigned long long ans=0;
    while(m--){
        int l=read()%n+1,r=read()%n+1;
        if (l>r)swap(l,r);
        ans+=qry(l,r);
    }
    cout<<ans;
    return 0;
}
```
[记录详情 - 洛谷 | 计算机科学教育新生态](https://www.luogu.com.cn/record/259839122)

Bonus: 这个做法也可以拓展到 O(n) - O(1) LCA，只要把欧拉序找出来然后求区间深度最小值对应的点的下标即可

参考：
[易懂科技：将LCA、RMQ、LA优化至理论最优复杂度 - 洛谷专栏](https://www.luogu.com.cn/article/kftl8q3s)
[最近公共祖先 | st 表求 lca 别用欧拉序了！！！ - skip2004 - 博客园](https://www.cnblogs.com/skip2004/p/12240164.html)

[推广一种常数较小，码量不大的 O(n)-O(1) RMQ - 洛谷专栏](https://www.luogu.com.cn/article/dmcrrb7k)
[一个RMQ问题的快速算法，以及区间众数 - 知乎](https://zhuanlan.zhihu.com/p/79423299)