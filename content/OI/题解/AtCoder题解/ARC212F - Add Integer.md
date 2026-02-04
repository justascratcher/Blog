---
tags:
  - 类欧
  - 思维/正难则反
---

[F - Add Integer](https://atcoder.jp/contests/arc212/tasks/arc212_f)

思路：正着推没啥用，但是如果已知结尾两项 $\{a,b\}$ 的话，那么能推出这两项的前两项只能是 $\{|a-b|,a\}$ ，那么直接从最后两项 $\{a,X\}$ 向回推。
注意到在保证 $p,q\neq 0$ 时， $|p-q|<\max\{p,q\}$ 所以只要枚举的 $a\leq M$ ，那么往回推时的序列就一定是合法的。
如果 $\{p,q\}$ 中有一个是 0，~~那么再怎样往回推，贡献都是 0，直接退出循环。~~ 其实是考虑模三的余数算答案。

接着如果 $p,q\neq 0$ 
假设 $p<q$，那么 q 可以写成 $q=kp+r \ (r<p)$
那么 $\{p,kp+r\}\to\{(k-1)p+r,p\}\to\{(k-2)p+r,(k-1)p+r\}\to\{p,(k-2)p+r\}\cdots$，每三次一循环，只有第一次的 $k=0$ 时，循环才会结束。
最后当 k 是奇数时，就会变成 $\{r,p\}$，操作了 $3\frac{k-1}{2}+1$ 次。
当 k 是偶数时，就变成 $\{p,r\}$，操作了 $3\frac{k}{2}$ 次。
中间计算了3*s次时为 $\{p,(k-2s)p+r\}$ 

假设 $p>q$，那么 p 可以写成 $p=kq+r \ (r<q)$
那么 $\{kq+r,q\}\to\{(k-1)q+r,kq+r\}\to\{q,(k-1)q+r\}\to\{(k-2)q+r,q\}\cdots$，每三次一循环，只有第一次的 $k=0$ 时，循环才会结束。
最后当 k 是奇数时，就会变成 $\{q,r\}$，操作了 $3\frac{k-1}{2}+2$ 次。
当 k 是偶数时，就变成 $\{r,q\}$，操作了 $3\frac{k}{2}$ 次。
中间计算了3*s次时为 $\{(k-2s)q+r,q\}$ 

然后剩余 $O(1)$ 次直接暴力算出。 

然后这玩意就是和辗转相除类似的东西，复杂度是 $O(\log n)$，枚举每个 $a$ 往回推就做完了。
~~不过这个太难写WA了~~事后调研发现是a的枚举范围写成了\[1-m\]，其实是\[0-m\]

其实回过头来看，就是每三次能把相对大对方两倍的数减去对方的两倍，
也就是 $p>2q$ 时， $\{p,q\}\to\{p-2q,q\} 每三次$
也就是 $q>2p$ 时， $\{p,q\}\to\{p,q-2p\}每三次$ 
其余情况下暴力计算后结果必然会回到这两种。
然后代码就好写了。

```cpp
const int mod=998244353;
int n,m,x;
int solve(int p,int q, int n){
    if(p==0||q==0) {
        if (q==0&&n%3==1){
            return p*p%mod;
        } else if (p==0&&n%3==2){
            return q*q%mod;
        }
        return 0;
    }
    if (n<6){
        For(i,1,n){
            int ans=abs(p-q);
            q=p,p=ans;
        }
        return p*q%mod;
    }
    if (p>q){
        int r=p%q,k=p/q,mk=n/3;
        if (2*mk > k) {
            if (k%2==0) return solve(r,q,n-3*(k/2));
            else return solve(q,r,n-3*((k-1)/2)-2);
        } else {
            return solve((k-2*mk)*q+r,q,n-3*(mk));
        }
    } else if (p<q){
        int r=q%p,k=q/p,mk=n/3;
        if (2*mk > k) {
            if (k%2==0) return solve(p,r,n-3*(k/2));
            else return solve(r,p,n-3*((k-1)/2)-1);
        } else {
            return solve(p,(k-2*mk)*p+r,n-3*(mk));
        }
    } else {
        return solve(0,p,n-1);
    }
}
signed main() {
    File(test);
    cin>>n>>m>>x;
    int ans=0;
    For(i,0,m){
        ans+=solve(i,x,n-2);
        ans%=mod;
        //cout << solve(i,x,n-2)<<endl;
    }
    cout<<ans<<endl;
    return 0;
}
```

simpler vesrion：
```cpp
const int mod=998244353;
int n,m,x;
int solve(int p,int q, int n){
    if(n==0)return p*q%mod;
    if (n < 4){
        return solve(abs(p-q),p,n-1);
    }	
    if (p==0||q==0){
        return solve(p,q,n%3);
    }
    if (q>3*p){
        int k=min((q-p)/(2*p),n/3);
        return solve(p,q-2*k*p,n-3*k);
    }
    return solve(abs(p-q),p,n-1);
}
signed main() {
    cin>>n>>m>>x;
    int ans=0;
    For(i,1,m){
        ans+=solve(i,x,n-2);
        ans%=mod;
        //cout << solve(i,x,n-2)<<endl;
    }
    cout<<ans<<endl;
    return 0;
}
```