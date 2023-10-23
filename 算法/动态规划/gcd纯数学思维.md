- https://codeforces.com/contest/1766/problem/D
- 题意找到连续的最长gcd(a+k，b+k) == 1(a < b,k = {0,1,2,...})
- 思路：
- gcd(a+k,b+k) == gcd(a+k,b - a)
1. a - b = 1时特判
2. 可以推出gcd(a+k,b+k) == gcd(a+k,b - a),具体证明见https://codeforces.com/blog/entry/110066
3. 设两个的结果分别为g和h，显然a+k和b+k都是g的倍数，那么让其中一个大的倍数减掉一个小的倍数，显然还是g的倍数，充分性成立
4. 反证法也同理，a+k和b-a肯定都是h的倍数，一个倍数加上另一个倍数，肯定还是h的倍数，必要性成立

- 接下来只需要找到b-a的质因子判断a+k是否有重复因子
- 一个一个加显然超时，我们可以先预处理出每个数字的最大质因子，然后利用传递性，得到关于b-a的所有质因子
- 设当前质因子为temp，更新答案的方法就是ans = min(ans,(temp - a%temp) % temp)

代码
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

const int N = 2e5+10,M = 1e7+10;
int to[M];
void solve() 
{
	int x,y;scanf("%d%d",&x,&y);
	if(y == x+1)
	{
		printf("-1\n");
		return ;
	}
	
	int now = y - x;
	int ans = 1e9;
	for(;now != 1;now /= to[now])
		ans = min(ans, (now - x % to[now]) % to[now]);
	
	printf("%d\n",ans);
}

signed main()
{
    /*
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    */

   	to[1] = 1;//每个数字到它最大的质因子
   	for(int i = 2;i < M;i ++)if(!to[i])
	{
		for(int j = i;j < M;j += i)to[j] = i;
	}

    int T = 1;
	scanf("%d",&T);
	//cin >> T;
    while(T--){
        solve();
    }
    return 0;
}

```
