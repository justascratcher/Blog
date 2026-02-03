---
tags:
  - 动态规划_DP
  - Hard/思路混乱
  - promotable_ovO
  - 不太懂
---

[F - Anti-DDoS](https://atcoder.jp/contests/abc301/tasks/abc301_f)

突然爬起来想到了这道题。

思路：
先考虑对于单个已知的串如何判定。
开一个 vis 数组，只保存大写字母的 vis ，当第一次重复 vis 一个字符时，它就进入了 DD 阶段，然后去找一个小写字母，就进入了 DDo 阶段，最后再找到一个大写字母的话就是 DDoS 串。
那么不是 DDoS 串的话，就也不是 DDo 串，更不是 DD 串。

考虑 DP，
令 $DD_i$ 表示到第 i 个字符时**前面长为 2 的子串里还没有 DD 串的方案数**，
令 $DDo_i$ 表示到第 i 个字符时**前面长为 3 的子串里还没有 DDo 串的方案数**，
令 $DDoS_i$ 表示到第 i 个字符时**前面长为 4 的子串里还没有 DDoS 串的方案数**，

>[!我的错误思路和正确思路的关系]
>错误思路：算反面，但反面太过诡异
>```mermaid
>graph TD
>	subgraph DD
>			DD_i
>		subgraph DDo
>				DDo_i
>			subgraph DDoS
>				DDoS_i
>			end
>		end
>	end
>
>```
>正确思路：算正面
>```mermaid
>graph TD
>	subgraph Not DDoS
>		DDoS_i
>		subgraph Not DDo
>			DDo_i
>			subgraph Not DD
>				DD_i
>			end
>		end
>	end
>
>```


那么对于非 DD 串，如果已经有出现一对的大写字母，那么 $DD_i=0$，否则就是所有 `?` 在剩下的没出现的字符里任取的方案数
非 DDo 串的话，如果下一个是小写，那么就 $DDo_i=DD_{i-1}$，如果是大写，那么 not DDo + U = not DDo, not DD + U may be = DD，但是  $DDo_i=DDo_{i-1}$

Bonus：
[DFA 和概率的解法](https://www.luogu.com.cn/article/4cvm9ycc)
[推广是否可行 - 洛谷](https://www.luogu.com.cn/discuss/1233035)（但还没有回答）

```cpp
mint fac[N],inv[N];
void init(int n) {
    fac[0]=1;
    For(i,1,n)fac[i]=fac[i-1]*i;
    inv[n]=qpow(fac[n],mod-2);
    rFor(i,0,n-1)inv[i]=inv[i+1]*(i+1);
}

mint C(int n,int m) {
    return fac[n]*inv[n-m]*inv[m];
}
mint A(int n,int m){
    return fac[n]*inv[n-m];
}
mint DD[N],DDo[N],DDoS[N];
int cnt,vis[26],flg=0,rst=26;
signed main() {
    File(test);
    string S;
    cin>>S;
    init(N-1);
    DD[0]=DDo[0]=DDoS[0]=1;
    For(i,1,S.size()){
        char c=S[i-1];
        if (c=='?'){
            cnt++;
            if (flg)DD[i]=0;
            else {
                for(int j=0;j<=rst&&j<=cnt;j++)
                    DD[i]+=qpow(26,cnt-j)*C(cnt,j)*A(rst,j);
            }
            DDo[i]=(DD[i-1]+DDo[i-1])*26;
            DDoS[i]=(DDoS[i-1]+DDo[i-1])*26;
        } else if ('a'<=c&&c<='z') {
            if (flg)DD[i]=0;
            else {
                for(int j=0;j<=rst&&j<=cnt;j++)
                    DD[i]+=qpow(26,cnt-j)*C(cnt,j)*A(rst,j);
            }
            DDo[i]=DD[i-1];
            DDoS[i]=DDoS[i-1];
        } else {
            if (vis[c-'A']) {
                flg=1;
            }
            else vis[c-'A']=1,rst--;
            if (flg)DD[i]=0;
            else {
                for(int j=0;j<=rst&&j<=cnt;j++)
                    DD[i]+=qpow(26,cnt-j)*C(cnt,j)*A(rst,j);
            }
            DDo[i]=DDo[i-1];
            DDoS[i]=DDo[i-1];
        }
    }
    cout<<DDoS[S.size()]<<endl;
    return 0;
}
```

参考：
[abc301_f Anti-DDoS - 洛谷专栏](https://www.luogu.com.cn/article/jy1pehdl)
[ABC301F - 洛谷专栏](https://www.luogu.com.cn/article/lnynpavi)