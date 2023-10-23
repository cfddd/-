- https://codeforces.com/contest/1762/problem/D
- 给一个长度为n的permutation，每次一个询问，得到结果为gcd（i，j），请在2*n次之内找到那个是0（或者哪两个之中之一是0）

- 思路
三个指针i，j，k（i<j<k）

令x=gcd(a[i],a[j]),y=gcd(a[i],a[k]);
1. 如果x==y，显然a[i]>0
2. 如果x<y，可以证明a[j]>0
3. 如果x>y，可以证明a[k]>0
- 这样就可以写出一个答案了

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
// #define int long long
using namespace std;

typedef long long LL;
typedef unsigned long long ULL;
typedef pair<int,int> PII;
typedef pair<LL,LL> PLL;

//#pragma GCC optimize(3,"Ofast","inline")
//#pragma GCC optimize(2)

const int N = 2e5+10,mod = 998244353;
bool st[N];

int ask(int a, int b) {
    cout << "? " << a << " " << b << endl;
    int ans = 0;
    cin >> ans;
    return ans;
}

void solve() 
{
    memset(st,0,sizeof st);
    int n;cin >> n;

    int a[3] = {1,2,3};
    for(;;)
    {
        sort(a,a+3);
        if(a[2] > n)break;
        int x = ask(a[0],a[1]),y = ask(a[0],a[2]);
        if(x == y)
        {
            st[a[0]] = 1;
            a[0] = a[2] + 1;
        }else if(x < y)
        {
            st[a[1]] = 1;
            a[1] = a[2] + 1;
        }else{
            st[a[2]] = 1;
            a[2] = a[2] + 1;
        }
    }

    cout << "! " << a[0] << " " << a[1] << endl;
    int t;cin >> t;
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
        solve();
    }
    return 0;
}

```
