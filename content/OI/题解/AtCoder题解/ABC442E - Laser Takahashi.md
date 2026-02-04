---
tags:
  - 计算几何/极角排序
---

[E - Laser Takahashi](https://atcoder.jp/contests/abc442/tasks/abc442_e)

思路：把怪兽按极角排序，然后的过程就显然了，如果 a < b，那就是 upper_bound b - lower_bound a，否则就是 n - (lower_bound a - upper_bound b) （画个图就出来了）
这里主要是极角排序

极角排序的思想是先把点按照半平面划分，其中 y+ 轴分给左边， y- 轴分给右边，然后在同一个半平面内的两个向量可以直接用叉乘判断关系（或者理解成算 tan ）

>[!TIP]
>这里的y轴的划分反过来也行，反正叉乘时负负得正，每一个半平面都相当于都是在右半平面，x = 0 的话，最后 y 的划分不同，就是 y 变号了，那正好乘出来与 0 的大小比较结果就反过来，正好是对的。或者用叉乘理解的话就是这样划分同一个半平面内叉乘 $\neq$ 0，那 y 的半轴分给谁都无所谓

>[!IDEA] 为什么不直接全用叉乘
>就是如果这样做，那么在极角 0 附近的正负关系就乱掉了，所以要分类
>点名批评 [极角排序常用方法 - 白雪儿 - 博客园](https://www.cnblogs.com/weixq351/p/9502025.html#:~:text=%E6%96%B9%E6%B3%951%EF%BC%9A%E5%88%A9%E7%94%A8atan2%EF%BC%88%EF%BC%89%E5%87%BD%E6%95%B0%E6%8C%89%E6%9E%81%E8%A7%92%E4%BB%8E%E5%B0%8F%E5%88%B0%E5%A4%A7%E6%8E%92%E5%BA%8F%E3%80%82%202%20%7B%203%20if%20%28atan2%28a.y%2Ca.x%29%21%3D%20atan2%28b.y%2Cb.x%29%29%204,atan2%28a.y%2Ca.x%29%3Catan2%28b.y%2Cb.x%29%3B%205%20else%20return%20a.x%3Cb.x%3B%206%20%7D%20%E6%96%B9%E6%B3%952%EF%BC%9A%E5%88%A9%E7%94%A8%E5%8F%89%E7%A7%AF%E6%8C%89%E6%9E%81%E8%A7%92%E4%BB%8E%E5%B0%8F%E5%88%B0%E5%A4%A7%E6%8E%92%E5%BA%8F%E3%80%82) 直接叉乘是错的

关键代码：
```cpp
auto dir=[](pii p) {
	if (p.l>0||p.r<0&&p.l==0) return 1;
	else return 0;
};
auto cmp=[&dir](pii l, pii r){
	auto d1=dir(l),d2=dir(r);
	if (d1<d2)return true;
	else if (d1>d2) return false;
	else {
		return l.r*r.l>r.r*l.l;
	}
};
```


```cpp
const int mod=998244353, N = 1e6; 
int lowbit(int u){return u&(-u);}
int n,q;
pair<pair<int,int>,int> m[N];
int idx[N];
signed main() {
    cin>>n>>q;
    For(i,1,n)cin>>m[i].l.l>>m[i].l.r,m[i].r=i;
    auto dir=[](pii p) {
        if (p.l>0||p.r<0&&p.l==0) return 1;
        else return 0;
    };
    auto cmp=[&dir](pii l, pii r){
        auto d1=dir(l),d2=dir(r);
        if (d1<d2)return true;
        else if (d1>d2) return false;
        else {
            return l.r*r.l>r.r*l.l;
        }
    };
    auto cmp2=[&cmp](pair<pii,int> sl, pair<pii,int> sr){
        return cmp(sl.l,sr.l);
    };
    sort(m+1,m+n+1,cmp2);
    For(i,1,n)idx[m[i].r]=i;
    while (q--) {
        int a,b;cin>>a>>b;
        a=lower_bound(m+1,m+n+1,m[idx[a]],cmp2)-m;
        b=upper_bound(m+1,m+n+1,m[idx[b]],cmp2)-m;
        if (a<b){
            cout<<b-a<<endl;
        } else {
            cout<<n-(a-b)<<endl;
        }
    }
}
```

参考：
[Editorial - AtCoder Beginner Contest 442 E](https://atcoder.jp/contests/abc442/editorial/15144)