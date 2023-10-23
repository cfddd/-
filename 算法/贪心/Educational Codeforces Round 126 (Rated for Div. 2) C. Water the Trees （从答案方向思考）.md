## 题目
- https://codeforces.com/contest/1661/problem/C
## 代码
```
#include<bits/stdc++.h>
#define debug1(a) cout<<#a<<'='<< a << endl;
#define debug2(a,b) cout<<#a<<" = "<<a<<"  "<<#b<<" = "<<b<<endl;
#define debug3(a,b,c) cout<<#a<<" = "<<a<<"  "<<#b<<" = "<<b<<"  "<<#c<<" = "<<c<<endl;
#define debug4(a,b,c,d) cout<<#a<<" = "<<a<<"  "<<#b<<" = "<<b<<"  "<<#c<<" = "<<c<<"  "<<#d<<" = "<<d<<endl;
#define debug5(a,b,c,d,e) cout<<#a<<" = "<<a<<"  "<<#b<<" = "<<b<<"  "<<#c<<" = "<<c<<"  "<<#d<<" = "<<d<<"  "<<#e<<" = "<<e<<endl;
#define endl "\n"
#define fi first
#define se second

#define int long long
//#define int __int128
using namespace std;

typedef long long LL;
typedef unsigned long long ULL;
typedef pair<int,int> PII;
typedef pair<LL,LL> PLL;

//#pragma GCC optimize(3,"Ofast","inline")
//#pragma GCC optimize(2)

//常数定义
const double eps = 1e-4;
const double PI = acos(-1.0);
const int INF = 0x3f3f3f3f;
const int N = 3e5+100;
int a[N];
int n;

int work(int u)
{
    int cnt1 = 0,cnt2 = 0;
    for(int i = 1;i <= n;i ++)
    {
        cnt1 += (u - a[i])&1;
        cnt2 += (u - a[i])>>1;
    }
    if(cnt1 > cnt2)return cnt1*2-1;
    else if(cnt1 == cnt2)return cnt2*2;
    else{
        int res = cnt1*2;
        cnt2 -= cnt1;
        res += cnt2*2/3*2 + ((cnt2*2) % 3);
        return res;
    }
}

void solve() 
{
    cin >> n;
    for(int i = 1;i <= n;i ++)cin >> a[i];

    int mx = *max_element(a+1,a+n+1);
    //debug1(mx);
    cout << min(work(mx),work(mx+1)) << endl;
}

signed main()
{
    
    /*
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    */
    int T = 1;cin >> T;

    while(T--){
        //puts(solve()?"YES":"NO");
        solve();
    }
    return 0;

}
/*

*/
```