- https://atcoder.jp/contests/abc259/tasks/abc259_f
- 树形dp（最简单的那种类型，但是题目的方法还是很巧妙的）
- 易知：负权边可以忽略

## 思路
### 定义
- 定义f[i][0]表示以i为根的子树尽量用到d[i]-1条边的最大可能（留一条边给父节点联通用）
- f[i][1]表示以i为根的子树尽量用到d[i]条边的最大可能
### 转移
- 就是那个最简单的树形dp，不能相邻放的
- f[i][0] = f[j][1],f[j][0] + w[idx]（j为前d[i]-1条最大的子路）
- f[i][1] = f[i][0] + (接着最大的一条边)

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
#define int long long
using namespace std;

typedef long long LL;
typedef unsigned long long ULL;
typedef pair<int,int> PII;
typedef pair<LL,LL> PLL;

//#pragma GCC optimize(3,"Ofast","inline")
//#pragma GCC optimize(2)

const int N = 3e5+10,M = N*2;
int h[N], e[M], ne[M], w[M], idx;
int d[N];
int f[N][2];
bool st[N];
void add(int a, int b,int c)  // 添加一条边a->b
{
    e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx ++ ;
}

void solve() 
{
    for(int i=0; i<N; i++)h[i] = -1;

    int n;cin >> n;
    for(int i = 1;i <= n;i ++)cin >> d[i];
    for(int i = 0; i < n-1; i++)
    {
        int a,b,c;cin >> a >> b >> c;
        if(c < 0)continue;
        add(a,b,c);
        add(b,a,c);
    }

    function<void(int,int)> dfs = [&](int u,int p)
    {
        st[u] = 1;
        
        vector<int> extra;
        int inist = 0;
        for(int i = h[u];i != -1;i = ne[i])
        {
            int j = e[i];
            if(j == p)continue;
            dfs(j,u);
            inist += f[j][1];
            if(d[j] && f[j][0] + w[i] - f[j][1] > 0)extra.push_back(f[j][0] + w[i] - f[j][1]);
        }
        sort(extra.begin(),extra.end(),greater<int>());
        
        f[u][0] = f[u][1] = inist;
        for(int i = 0;i < min(d[u],(int)extra.size());i ++)
        {
            f[u][0] = f[u][1];
            f[u][1] += extra[i];
        }
        if(d[u] > (int)extra.size())f[u][0] = f[u][1];
        
    };

    int ans = 0;
    for(int i = 1;i <= n;i ++)
        if(!st[i]){
            
            dfs(i,-1);
            //debug3(i,f[i][1],f[i][0]);
            ans += max(f[i][0],f[i][1]);
        }

    cout << ans << endl;
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

*/

```
