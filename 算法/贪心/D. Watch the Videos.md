- https://codeforces.com/problemset/problem/1765/D

观察找规律

最大的和二分最小的，然后交替相邻，看是否合法

找到最长的合法段

答案就是总长度减去合法段加上原先的长度

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

const int N = 2e5+10;
int a[N];
int n,v;

bool check(int x)
{
    int l = 1,r = x;
    while(l < r)
    {
        //debug2(l,r);
        if(a[l] + a[r] > v)return false;
        r --;
        if(l < r && a[l] + a[r] > v)return false;
        l ++;
    }
    return true;
}

void solve() 
{
    cin >> n >> v;
    int sum = 0;
    for(int i = 1;i <= n;i ++){
        cin >> a[i];
        sum += a[i];
    }
    
    sort(a+1,a+n+1);
    
    int l = 1,r = n;
    int mid;
    while(l < r)
    {
        mid = l+r+1>> 1;
        //debug3(l,r,mid);
        if(check(mid))l = mid;
        else r = mid-1;
        
    }
    
    cout << sum + n - l + 1<< endl;
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
