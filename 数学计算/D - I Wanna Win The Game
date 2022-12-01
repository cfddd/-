https://atcoder.jp/contests/arc116/tasks/arc116_d
```
#include<bits/stdc++.h>
#define debug1(a) cout<<#a<<'='<< a << endl;
#define debug2(a,b) cout<<#a<<" = "<<a<<"  "<<#b<<" = "<<b<<endl;
#define debug3(a,b,c) cout<<#a<<" = "<<a<<"  "<<#b<<" = "<<b<<"  "<<#c<<" = "<<c<<endl;
#define debug4(a,b,c,d) cout<<#a<<" = "<<a<<"  "<<#b<<" = "<<b<<"  "<<#c<<" = "<<c<<"  "<<#d<<" = "<<d<<endl;
#define debug5(a,b,c,d,e) cout<<#a<<" = "<<a<<"  "<<#b<<" = "<<b<<"  "<<#c<<" = "<<c<<"  "<<#d<<" = "<<d<<"  "<<#e<<" = "<<e<<endl;
#define debug0(x) cout << "debug0: " << x << endl
#define fr(t, i, n)for (long long i = t; i < n; i++)
#define YES cout<<"Yes"<<endl
#define NO cout<<"No"<<endl
#define fi first
#define se second
#define int long long
using namespace std;

typedef long long LL;
typedef unsigned long long ULL;
typedef pair<int,int> PII;
typedef pair<LL,LL> PLL;

//#pragma GCC optimize(3,"Ofast","inline")
//#pragma GCC optimize(2)

const int N = 5e3+10,mod = 998244353;
int C[N][N];
int f[N];
/*
求异或和为0，那么必然按位来看
按位dp
f[i]表示和为i且异或和为0时，方案数量
从低位开始，枚举当前最低位的1的数量（必须是偶数个1，显然奇数个对答案不成立）

左边除了最低位得那些位
f[i] += f[(i-j)/2]*C[n][j]


*/
void solve() 
{
    int n,m;cin >> n >> m;
    C[1][0] = C[1][1] = 1;
    for(int i = 2;i <= n;i ++)
    {
        for(int j = 0;j <= i;j ++)
        {
            if(j)C[i][j] = (C[i-1][j] + C[i-1][j-1]) % mod;
            else C[i][j] = 1;
        }
    }
    f[0] = 1;
    for(int i = 2;i <= m;i += 2)//数字范围
    {
        for(int j = 0;j <= n && j <= i;j += 2)//位数范围
        {
            f[i] = (f[i] + f[(i-j)/2]*C[n][j]) % mod;
        }
    }
    cout << f[m] << endl;
}

signed main()
{
    /*
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    */

    int T = 1;//cin >> T;
    while(T--){
        solve();
    }
    return 0;
}

```
