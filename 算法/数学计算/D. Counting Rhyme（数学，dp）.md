##  题意
给你一个整数数组 $a_1, a_2, \ldots, a_n$ 。

如果**不存在**个整数 $k$ ( $1 \le k \le n$ )，使得 $a_i$ 能被 $a_k$ 整除，且 $a_j$ 能同时被 $a_k$ 整除，则一对整数 $(i, j)$ （如 $1\le i \lt j \le n$）称为好整数。

请找出好数对的个数。
## 思路
请注意，如果 $a_i$ 能被 $a_k$ 整除， $a_j$ 能被 $a_k$ 整除，那么 $\gcd(a_i, a_j)$ 也能被 $a_k$ 整除。我们来计算数组中每个 $g$ 的对数，这样 $\gcd$ 就等于 $g$ 。

为此，我们将创建 $dp_g$ ——表示 $\gcd$ 等于 $g$ 的数组对数。
为此，我们需要一个计数数组 $cnt_x$ ，即数组中数字 $x$ 的出现次数。要计算 $dp_g$ ，我们需要知道有多少个数字能被 $g$ 整除，特别是和 $s = cnt_g + cnt_{2g} + \ldots + cnt_{kg}$ ，其中 $kg \leq n$ 。
因此， $\gcd$ 等于 $g$ 的数对的个数是 $\frac{s \cdot (s - 1)}{2}$ 。但是，并不是所有的 $\gcd$ 都等于 $g$ ，它可以等于 $g$ 的倍数。因此， $dp_g = \frac{s \cdot (s - 1)}{2} - dp_{2g} - \ldots - dp_{kg}$ 。这种动态程序设计可以在 $O(n \log n)$ 时间内完成计算。

现在我们来了解一下哪些 $\gcd$ 值不合适。如果数组中有一个数字 $x$ （即 $cnt_x \gt 0$ ），那么所有 $g$ 的倍数 $x$ 都不合适。这也可以在 $O(n \log n)$ 时间内计算出来。最后，我们只需将所有 $dp_g$ 值求和，其中数字 $g$ 不能被数组中的任何数字整除。
## 代码
```cpp
const int N = 1e6+10;

int st[N];
int cnt[N];
int dp[N];

void solve()
{
	int n;cin >> n;
	for(int i = 0;i < n;i ++){
		int x;cin >> x;
		cnt[x] ++;
		st[x] = 1;
	}
	
	int ans = 0;
	
	for(int i = n;i >= 1;i --)
	{
		int sum = 0;
		for(int j = i;j <= n;j += i)
		{
			sum += cnt[j];
		}
		dp[i] = sum * (sum - 1) / 2;
		for(int j = i + i;j <= n;j += i)
		{
			dp[i] -= dp[j];
			st[j] |= st[i];
		}
	}
	
	for(int i=1;i<=n;++i)ans+=(st[i]==0)*dp[i];
	
	for(int i = 0;i <= n;i ++)
	{
		st[i] = dp[i] = cnt[i] = 0;
	}
	
	// debug1(ans);
	cout << ans << endl;
}
```