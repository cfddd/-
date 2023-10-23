## 题目
- https://codeforces.com/problemset/problem/1054/D
## 题意

- 输入正整数 n(≤2e5) k(≤30) 和长为 n 的数组 a(0≤a[i]≤pow(2,k)-1)。

- 设 mask = (1<<k)-1，每次操作你可以把任意 a[i] 修改为 a[i] XOR mask，你可以操作任意次（包括 0 次）。
- 修改后，最多有多少个 a 的非空连续子数组，其异或和不等于 0？
## 思路
- 找最多的不为 0 ，不方便，转化为找最少的为0
- 需要从异或前缀和出发，找一段连续子数组异或和为 0 非常方便，$\sum C_{前缀异或和某数字出现的次数}^2$
- 修改一个数字为 a[i] XOR mask，那么前缀异或i后面都异或mask，所以就变为一段一段奇数为 a[i] 偶数为 a[i] XOR mask的数组
- 再进一步，想要最灵活的修改数组，其实一个一个也可以
- 最后，只要让 a[i] 和 a[i] XOR mask 数量平均就可以实现最小（简单数学）
- 不要忘记在前缀意义上的起点 0 ，初始时 cnt[0] = 1
## 代码
```
const int N = 2e5+10;
int a[N];
void solve()
{
	int n,k;cin >> n >> k;
	for(int i = 1;i <= n;i ++)cin >> a[i],a[i] ^= a[i-1];
	const int mask = (1 << k) - 1;

	map<int,int> cnt;
	for(int i = 0;i <= n;i ++)
	{
		int flip = a[i] ^ mask;
		if(cnt[a[i]] <= cnt[flip])cnt[a[i]] ++;
		else cnt[flip] ++;
	}

	LL ans = 0;
	for(auto [x,y]:cnt)
	{
		ans += 1LL*y*(y-1)/2;
		// debug2(x,y);
	}

	cout << 1LL*n*(n+1)/2 - ans << endl;
}
```