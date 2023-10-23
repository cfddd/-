```
#include <bits/stdc++.h>
#define debug1(a) cout << #a << '=' << a << endl;
#define debug2(a, b) cout << #a << " = " << a << "  " << #b << " = " << b << endl;
#define debug3(a, b, c) cout << #a << " = " << a << "  " << #b << " = " << b << "  " << #c << " = " << c << endl;
#define debug4(a, b, c, d) cout << #a << " = " << a << "  " << #b << " = " << b << "  " << #c << " = " << c << "  " << #d << " = " << d << endl;
#define debug5(a, b, c, d, e) cout << #a << " = " << a << "  " << #b << " = " << b << "  " << #c << " = " << c << "  " << #d << " = " << d << "  " << #e << " = " << e << endl;
#define endl "\n"
#define fi first
#define se second
#define caseT \
	int T;    \
	cin >> T; \
	while (T--)
// #define int long long
// #define int __int128
using namespace std;

typedef long long LL;
typedef unsigned long long ULL;
typedef pair<int, int> PII;
typedef pair<LL, LL> PLL;

// #pragma GCC optimize(3,"Ofast","inline")
// #pragma GCC optimize(2)

inline int rd()
{
	int f = 1, x = 0;
	char c = getchar();
	while (c < '0' || '9' < c)
	{
		if (c == '-')
			f = -1;
		c = getchar();
	}
	while ('0' <= c && c <= '9')
		x = (x << 3) + (x << 1) + (c ^ 48), c = getchar();
	return f * x;
}

// 常数定义
const double eps = 1e-6;
const double PI = acos(-1.0);
const int INF = 0x3f3f3f3f;
const int N = 1e5 + 10, M = 2 * N;
int h[N], e[M], ne[M], idx;
bool affected[N];
int dist1[N], dist2[N];

int n, m, d;
int ans;

void add(int a, int b)
{
	e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void dfs1(int u, int father)
{
	if(affected[u])dist1[u] = 0;

	int &d1 = dist1[u], &d2 = dist2[u];

	for (int i = h[u]; i != -1; i = ne[i])
	{
		int v = e[i];
		if (v == father)
			continue;
			
		dfs1(v, u);
		int d = dist1[v] + 1;
		// debug1(d);
		if(dist1[v] == -1)continue;



		if (d >= d1)
			d2 = d1, d1 = d;
		else if (d > d2)
			d2 = d;
	}

	// return dist;
}

void dfs2(int u, int fa,int preLen)
{
	if(max(dist1[u],preLen) <= d)ans ++;
	// debug2(u,max(dist1[u],preLen));
	for (int i = h[u]; i != -1; i = ne[i])
	{
		int v = e[i];
		if (v == fa)
			continue;
		
		int Len = preLen;
		if(dist1[v] >= 0)Len = (dist1[v] == dist1[u]-1)?max(Len,dist2[u])+1:max(Len,dist1[u])+1;
		else Len = max(Len,dist1[u]) + 1;
		
		dfs2(v, u, Len);

	}
}

void solve()
{
	memset(h, -1, sizeof h);
	memset(dist1,-0x3f,sizeof dist1);
	memset(dist2,-0x3f,sizeof dist2);
	
	// memset(f, -1, sizeof f);

	cin >> n >> m >> d;
	for (int i = 0; i < m; i++)
	{
		int t;
		cin >> t;
		affected[t] = 1;
	}

	for (int i = 0; i < n - 1; i++)
	{
		int x, y;
		cin >> x >> y;
		add(x, y);
		add(y, x);
	}

	dfs1(1, -1);

	dfs2(1, -1, -1044266559);
	// for(int i = 1;i <= n;i ++)debug3(i,dist1[i],dist2[i]);
	cout << ans << endl;
}

signed main()
{

	ios::sync_with_stdio(false);
	cin.tie(0);
	cout.tie(0);

	// caseT
	solve();

	return 0;
}
/*

*/

```