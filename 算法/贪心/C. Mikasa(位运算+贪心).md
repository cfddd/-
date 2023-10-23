## 题目
- https://codeforces.com/problemset/problem/1554/C
## 题意
- 输入 t(≤3e4) 表示 t 组数据，每组数据输入两个整数 n 和 m，均在 [0,1e9] 范围内。
- 定义数组 a = [n xor 0, n xor 1, n xor 2, ..., n xor m]。
- 输出mex(a)
## 思路
- 答案存在0 ~ m+1之间
- 设答案为 x ，那么必然存在一个y在0 ~ m之间，使得 n ^ y == x
- 因为n ^ y == x <=> n ^ x == y，得到结论n ^ x <= m
- 构造出能够使n ^ x <= m成立的最大x
- 从高位往低位，只有n为0，m为1的位置才可以让x为1（同时n < m才有解）
## 代码
```

void solve() 
{
	int n,m;cin >> n >> m;
	if(n > m)
	{
		cout << 0 << endl;
		return ;
	}
	int ans = 0;
	m ++;
	for(int i = 30;i >= 0 && n < m;i --)
	{
		if((n >> i & 1) < (m >> i & 1))
		{
			ans |= 1 << i;
			n |= 1 << i;
		}
	}
	cout << ans << endl;
}
```