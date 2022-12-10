# https://atcoder.jp/contests/arc132/tasks/arc132_c

- 很难想到的动态规划，优化空间的思路非常巧妙
- 用相对位置来转移
- f[i][j]表示i之前，放置数字的压缩情况为j，的所有方案数
- ** f[i+1][(j | (1 << k)) >> 1] += f[i][j] **
- k表示i放的数字的相对位置
- 具体转移还是看代码
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
#define nO cout<<"no"<<endl
#define fi first
#define se second
//#define int long long
using namespace std;

typedef long long LL;
typedef unsigned long long ULL;
typedef pair<int,int> PII;
typedef pair<LL,LL> PLL;

//#pragma GCC optimize(3,"Ofast","inline")
//#pragma GCC optimize(2)

const int N = 510,mod = 998244353;
int f[N][1<<15];                                //表示i之前的所有数字的使用情况，压缩为j
int a[N];                                       //通过相对位置来转移
                                                //因为像个的太远肯定彼此之间不会用到
void solve() 
{
    f[0][0] = 1;                    
    int n,d;cin >> n >> d;
    for(int i = 0;i < n;i ++)
    {
        cin >> a[i];
        a[i] --;
    }

    for(int i = 0;i < n;i ++)                   //位置
    {
        for(int j = 0;j < 1<<(2*d + 1);j ++)    //状态
        {

            if(a[i] >= 0)                       //a[i]大于0的情况
            {
                int temp = a[i] - i + d;        //已经放的数字对应的相对位置
                if(~j>>temp&1)(f[i+1][(j | (1<<temp)) >> 1] += f[i][j]) %= mod;
                continue;
            }

            for(int k = 0;k < 2*d+1;k ++)       //a[i]是-1的情况
            {
                int newi = k - d + i;
                if(newi < 0 || newi >= n)continue;
                if(j>>k&1)continue;             
                (f[i+1][(j|(1<<k)) >> 1] += f[i][j]) %= mod;
            }
        }
    }
    // for(int i = 0;i <= n;i ++)                   //位置
    // {
    //     debug2(i,f[i][(1<<d) - 1]);
    // }
    cout << f[n][((1<<d) - 1)] << endl;             //((1<<d) - 1)d个1的二进制，正好是n之前所有的
    
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
