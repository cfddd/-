## 题目
- https://codeforces.com/problemset/problem/1661/D
## 题意
- 给一个长度为 n 的数组 a，和一个正数k，每次在数组 a 中选取连续的k个元素
- 每个元素减去1，2，3……k
- 问至少要多少次操作，才能呢使数组 a 中所有数字小于 0 
## 思路
- 从后往前贪心，找到第一个大于 0 的位置，通过一定操作使得这个元素小于 0 
- 可以维护一个差分数组，前k个位置 - 1，最后一个元素 + k，操作就变为区间元素修改
- 如何找到这个位置？可以用树状数组或者限度按时来实现这个[经典功能](https://www.cnblogs.com/cfddfc/p/17065029.html#%E4%BB%A3%E7%A0%81%E5%AE%9E%E7%8E%B0)
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
LL a[N],b[N];
LL x[N],neg[N];
LL n,k;

LL lowbit(LL x)  // 返回末尾的1
{
    return x & -x;
}

void add(LL x,LL w){
    if(x == 0)return ;
    for(LL i = x;i <= n+50;i += lowbit(i)){ //同时维护两个树状数组
        a[i] += w;
        b[i] += x*w;
    }
}

LL query(LL u){
    LL res = 0;
    //if(u == 0)return res;
    for(LL i = u;i > 0;i -= lowbit(i)){ //优化的区间查询方案
        res += (u+1) * a[i] - b[i];     //计算公式
    }
    return res;
}

void solve() 
{
    cin >> n >> k;
    for(int i = 2;i <= n+1;i ++)cin >> x[i];
  
    int ans = 0;
    for(int r = n+1;r > k+1;r --)
    {
        int l = r-k+1;
        int now = query(n+2) - query(r-1);
        if(now < x[r])
        {
            int cnt = (x[r] - now) / k + (((x[r] - now) % k) > 0);

            add(r,cnt*k);add(r+1,-1*cnt*k);
            add(l-1,-1*cnt);add(r,cnt);
            ans += cnt;
        }
    }

    int mx = 0;
    for(int r = k+1;r >= 2;r --)
    {
        int l = 1;
        int now = query(n+2) - query(r-1);
        if(now < x[r])
        {
            int cnt = (x[r] - now) / (r-1) + (((x[r] - now) % (r-1)) > 0);
            mx = max(mx,cnt);
        }
    }

    cout << ans + mx << endl;
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
/*
7 7
50 17 81 25 42 39 96

*/
```
## 正解思路
- 在差分数组的基础上再次进行差分，这一段的每次的操作就变为-1，k-1
- 暂时
