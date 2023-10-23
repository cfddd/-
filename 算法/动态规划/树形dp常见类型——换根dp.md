## 题目
1. https://codeforces.com/problemset/problem/219/D
2. https://leetcode.cn/problems/count-number-of-possible-root-nodes/
3. https://leetcode.cn/problems/minimum-height-trees/description/、
4. 树的直径也算类似
## 题意（仅第一题）
- 给n-1条**有向边**，构成一棵树
- 从一个节点出发，到达所有点，最少要翻转多少次边
- 输出这个最小的数字，然后输出所有可能的起点
## 思路
- 因为是一棵树，不考虑边的方向，那么任意一个点当根节点都可以到达其他所有点
- 所以我们可以建一个有权树，一条已有的边（u -> v）权为0，那么反向边（v -> u）边权为1
- 剩下的就是换根dp这个套路的
  - 定义f[u]为以u为根的方案数，通过一次普通的搜索就可以算出来
  - 再次搜索，同时携带根节点的一些信息提供给子节点计算
  - 具体细节见代码
## 代码

```
const int N = 2e5+10,M = 2*N;

int e[M],ne[M],h[N],w[M],idx;
int f[N];
int ans[N];

void add(int a,int b,int c)
{
	e[idx] = b,w[idx] = c,ne[idx] = h[a],h[a] = idx ++;
}

int dfs1(int u,int fa)
{
	for(int i = h[u];i != -1;i = ne[i])
	{
		int v = e[i];
		if(v == fa)continue;
		f[u] += dfs1(v,u) + w[i];
	}

	return f[u];
}

void dfs2(int u,int fa,int pre)
{
	int sum = f[u];
	ans[u] = pre + sum;
	for(int i = h[u];i != -1;i = ne[i])
	{
		int v = e[i];
		if(v == fa)continue;

		int add = w[i] == 0?1:-1;//通用的变化
		dfs2(v,u,pre + sum - f[v] + add);
	}
}

void solve()
{
	memset(h,-1,sizeof h);
	int n;cin >> n;
	for(int i = 0;i < n-1;i ++)
	{
		int a,b;cin >> a >> b;
		add(a,b,0);
		add(b,a,1);
	}

	dfs1(1,-1);

	dfs2(1,-1,0);

	int mi = *min_element(ans+1,ans+n+1);
	cout << mi << endl;
	// for(int i = 1;i <= n;i ++)debug2(i,ans[i]);
	for(int i = 1;i <= n;i ++)if(ans[i] == mi)cout << i << " ";
	cout << endl;
}

```