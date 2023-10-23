## 题目
- https://codeforces.com/contest/1379/problem/C
## 题意
- 输入 t(≤1e4) 表示 t 组数据。所有数据的 m 之和 ≤1e5。

- 每组数据输入 n(≤1e9) m(≤1e5) 表示有 m 种物品，每种物品有无限个，你需要选择 n 个。
- 然后输入 m 行，每行两个数字 a[i] 和 b[i]，范围在 [0,1e9]。

- 如果第 i 种物品选 x 个（x>0），收益为 a[i]+(x-1)*b[i]（不是从1~x）
- 输出最大收益。
## 思路
- 最多只有一种花被选择了两次及以上
- 如果一种花需要被选择两次及以上，那么在第二步的时候可以发现先选择a[j] > a[i]的花一次更优
- 如果有多余的，可以再留下来选当前这个
- 排序+前缀和求出选一次的花的a[i]，二分找左边界
## 代码
```cpp
const int N = 1e5+10;
PII a[N];
int sum[N];

void solve()
{
	int n,m;cin >> n >> m;
	for(int i = 1;i <= m;i ++)
		cin >> a[i].fi >> a[i].se;
	
	sort(a+1,a+m+1);

	for(int i = 1;i <= m;i ++)
		sum[i] = sum[i-1] + a[i].fi;

	int ans = 0;
	for(int i = 1;i <= m;i ++)
	{
		int id = lower_bound(a+1,a+m+1,make_pair(a[i].se,-1),[&](PII x,PII y)
		{
			return x.fi < y.fi;
		}) - a;
		
		int left = n - (m - id + 1);
		// debug2(id,left);
		if(left <= 0)
		{
			ans = max(ans,sum[m] - sum[m - n]);
		}else{
			ans = max(ans,sum[m] - sum[id-1] + (left-1 + (id <= i))*a[i].se + (id > i?a[i].fi:0));
			// debug1(sum[m] - sum[id-1]);
		}
		// debug1(ans);
	}

	cout << ans << endl;
}

```