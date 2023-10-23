## 题目
- https://codeforces.com/contest/1804/problem/D
## 题意
- 给 n 个长度为 m 的 01 字符串，m mod 4 = 0
- 把字符串切割成 m/2 个单个字符和 m/4 个连续的两个字符
- 切割后的字符串中存在 1 ，价值 +1
- 问最大切割价值和最小切割价值
## 思路
### 最小
- 找出尽可能多的 11 ，放在一起
- 如果数量大于等于 m/4 ，那么之后不会有节约掉一个 1 的机会
- 如果数量小于 m/4 ，节约掉了一个 1 
### 最大
- 找出尽可能多的 非11，帮这些放在一起
- 如果数量大于等于 m/4 ，说明每个 1 都可以开成一个房间
- 如果数量小于 m/4 ，就要牺牲 11 来协调
## 代码
```
void solve()
{
	int n,m;cin >> n >> m;
	int ans1 = 0,ans2 = 0;
	vector<string> s(n);
	vector<int> cnt1(n,0);
	for(int i = 0;i < n;i ++)
	{
		cin >> s[i];
		for(int j = 0;j < m;j ++)
			if(s[i][j] == '1')cnt1[i] ++;
	}

	
	for(int i = 0;i < n;i ++)
	{
		int res1 = 0;
		for(int j = 0;j < m-1;j ++)
		{
			if(s[i][j] == s[i][j+1] && s[i][j] == '1')res1 ++,j ++;
			if(res1 >= m/4)
				break;
		}
		ans1 += cnt1[i] - res1*2 + res1;
	}

	for(int i = 0;i < n;i ++)
	{
		int res1 = 0;
		for(int j = 0;j < m-1;j ++)
		{
			if(s[i][j] != '1' || s[i][j+1] != '1')
			{
				res1 ++,j ++;
			}
		}
		if(res1 >= m/4)ans2 += cnt1[i];
		else{
			ans2 += cnt1[i] - (m/4 - res1);
		}
	}
	cout << ans1 << " " << ans2 << endl;
}
```