---
tags:
  - 思维/数据维护
  - 数据结构/线段树
---

[G - Takoyaki and Flip](https://atcoder.jp/contests/abc441/tasks/abc441_g)

就是线段树，但是要注意懒标记的更新顺序，还有一些思路问题
核心：
```cpp
struct S {
    int flp,clr,add,l,r,mx,up;
} tr[4*N];

void pushdown(S&f,S&l,S&r){
	// 顺序很关键
    if (f.flp){
        f.flp=0;
        l.flp^=1; r.flp^=1;
        l.up=(l.r-l.l+1-l.up);
        r.up=(r.r-r.l+1-r.up);
        if (l.up==0)l.mx=0;
        if (r.up==0)r.mx=0;
    }
    if (f.clr){
        l.add=0,r.add=0;
        l.clr=1,r.clr=1;
        f.clr=0;
        l.mx=0,r.mx=0;
    }
    if (f.add){
        l.add+=f.add,r.add+=f.add;
        if (l.up) l.mx+=f.add; // <
        if (r.up) r.mx+=f.add; // <
        f.add=0;
    }
}

void pushup(S&f,S&l,S&r){
    f.mx=max(l.mx,r.mx);
    f.up=l.up+r.up;
}
```

```cpp
const int mod=998244353, N = 2e5+5;
int lowbit(int u) {return u&(-u);}
int qpow(int a,int b){
    if (b==0)return 1;
    int v=qpow(a,b>>1);
    return (ll)v*v%mod*((b&1)?a:1)%mod;
}
struct S {
    int flp,clr,add,l,r,mx,up;
} tr[4*N];

void pushdown(S&f,S&l,S&r){
    if (f.flp){
        f.flp=0;
        l.flp^=1; r.flp^=1;
        l.up=(l.r-l.l+1-l.up);
        r.up=(r.r-r.l+1-r.up);
        if (l.up==0)l.mx=0;
        if (r.up==0)r.mx=0;
    }
    if (f.clr){
        l.add=0,r.add=0;
        l.clr=1,r.clr=1;
        f.clr=0;
        l.mx=0,r.mx=0;
    }
    if (f.add){
        l.add+=f.add,r.add+=f.add;
        if (l.up) l.mx+=f.add;
        if (r.up) r.mx+=f.add;
        f.add=0;
    }
}

void pushup(S&f,S&l,S&r){
    f.mx=max(l.mx,r.mx);
    f.up=l.up+r.up;
}

void pushdown(int u){
    pushdown(tr[u],tr[u*2],tr[u*2|1]);
}

void pushup(int u){
    pushup(tr[u],tr[u*2],tr[u*2|1]);
}

void build(int l,int r,int u=1){
    tr[u]={0,0,0,l,r,0,r-l+1};
    if(l==r){
        return;
    }
    int mid=(l+r)>>1;
    build(l,mid,u*2);
    build(mid+1,r,u*2|1);
}
#define ND auto&nd=tr[u]
#define MID int mid=(nd.l+nd.r)>>1;
void add(int l, int r, int v, int u=1){
    ND;
    if (nd.r<l||r<nd.l)return;
    if(l<=nd.l&&nd.r<=r){
        nd.add+=v;
        if (nd.up) nd.mx+=v;
        return;
    }
    MID;
    pushdown(u);
    if(l<=mid)add(l,r,v,u*2);
    if(mid<r)add(l,r,v,u*2|1);
    pushup(u);
}

void flip(int l, int r,int u=1){
    ND;
    if (nd.r<l||r<nd.l)return;
    if(l<=nd.l&&nd.r<=r){
        nd.clr=1;
        nd.flp^=1;
        nd.add=0;
        nd.up=(nd.r-nd.l+1)-nd.up;
        nd.mx=0;
        return;
    }
    MID;
    pushdown(u);
    if(l<=mid)flip(l,r,u*2);
    if(mid<r)flip(l,r,u*2|1);
    pushup(u);
}

int qry(int l, int r, int u=1){
    ND;
    if (nd.r<l||r<nd.l)return 0;
    if(l<=nd.l&&nd.r<=r){
        return nd.mx;
    }
    MID;
    pushdown(u);
    int mx=0;
    if(l<=mid)mx=qry(l,r,u*2);
    if(mid<r)mx=max(mx,qry(l,r,u*2|1));
    return mx;
}

signed main() {
    File(test);
    int n,q;cin>>n>>q;
    build(1,n);
    while(q--){
        int t,l,r;cin>>t>>l>>r;
        switch(t){
        case 1:{
            int x;cin>>x;
            add(l,r,x);
            break;
        }
        case 2:
            flip(l,r);
            break;
        case 3:
            cout<<qry(l,r)<<endl;
            break;
        }
    }
    return 0;
}
```