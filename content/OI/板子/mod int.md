
```cpp
struct mint {
    int v;
    mint(int v=0):v(v){}
    mint operator-(){return -v;}
    friend mint operator+(mint a, mint b){return (a.v+b.v)%mod;}
    friend mint operator-(mint a, mint b){return (a.v-b.v)%mod;}
    friend mint operator*(mint a, mint b){return (ll)a.v*b.v%mod;}
    friend mint operator/(mint a, mint b){return (ll)a.v*qpow(b.v,mod-2)%mod;}
    mint& operator +=(mint b){return (*this)=(*this+b);}
    mint& operator -=(mint b){return (*this)=(*this-b);}
    mint& operator *=(mint b){return (*this)=(*this*b);}
    mint& operator /=(mint b){return (*this)=(*this/b);}
    bool operator ==(mint b){return v==b.v;}
    bool operator !=(mint b){return v!=b.v;}
    bool operator >=(mint b){return v>=b.v;}
    bool operator <=(mint b){return v<=b.v;}
    bool operator >(mint b){return v>b.v;}
    bool operator <(mint b){return v<b.v;}
    mint operator >>(mint b){return v>>b.v;}
    mint operator <<(mint b){return (v<<b.v)%mod;}    
    friend ostream& operator<<(ostream&os,mint v){
	    return os<<v.reg();
    }
    friend istream& operator>>(istream&os,mint v){
	    return os>>v.v;
    }
    int reg()const{
        return (v+mod)%mod;
    }
    explicit operator int() {
        return v;
    }
};

mint qpow(mint a,mint b){
    if (b==0)return 1;
    mint v=qpow(a,b>>1);
    return v*v*((int(b)&1)?a:1);
}
```
