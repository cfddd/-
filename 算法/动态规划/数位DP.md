> [算法学习笔记(22)：数位DP（数位动态规划） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/613107701)

## 例题
### 数字游戏
这种数字必须满足从左到右各位数字呈非下降关系，如 123123，446446。

现在大家决定玩一个游戏，指定一个整数闭区间 [a,b][�,�]，问这个区间内有多少个不降数。

```cpp
const int N = 13;
int A[N],cnt;
int dp[N][N];	//第i个数字，上一个填的是什么

int dfs(int pos,int last,int limit)
{
	
	if(pos == cnt)
		return 1;
	
	if(!limit && dp[pos][last] != -1)return dp[pos][last];      //没被限制，当前位才可以通过上一位记忆化处理掉
	
	int ans = 0;
	
	int up = limit ? A[pos] : 9;
	for(int i = last;i <= up;i ++)
	{
		ans += dfs(pos+1,i,limit && i == up);
	}
	
	return dp[pos][last] = ans;
}

LL f(LL x)
{
    cnt = 0;
    memset(dp, -1, sizeof(dp));
    memset(A, 0, sizeof(A));
    while (x)
        A[cnt++] = x % 10, x /= 10;
    reverse(A, A + cnt);
    return dfs(0, 0, true);
}

void solve()
{
	int a,b;
	while(cin >> a >> b)
	{
		cout << f(b) - f(a-1) << endl;
	}
}
```
#### 数字游戏Ⅱ
```cpp
#include <bits/stdc++.h>

using namespace std;

typedef long long LL;
typedef unsigned long long ULL;
typedef pair<int, int> PII;
typedef pair<LL, LL> PLL;

const double PI = acos(-1.0);
const int N = 15,M = 1010;
int A[N],cnt;
int dp[N][M];	//第i个数字，上一个填的和为sum，后面没被限制的合法数量
int mod;
int dfs(int pos,int sum,int limit)
{
	if(pos == cnt)
		return sum % mod == 0;
	
	if(!limit && dp[pos][sum] != -1)
	{
		return dp[pos][sum];      //没被限制，当前位才可以通过上一位记忆化处理掉
	}
	
	int ans = 0;
	
	int up = limit ? A[pos] : 9;
	for(int i = 0;i <= up;i ++)
	{
    	ans += dfs(pos+1,sum + i,limit && i == up);
	}
	
	if(limit)		//被限制，或者也没有前导0（限制），返回值即可
	{
		return ans;
	}
	return dp[pos][sum] = ans;
}

LL f(LL x)
{
    cnt = 0;
    memset(dp, -1, sizeof(dp));

    while (x)
        A[cnt++] = x % 10, x /= 10;
    
    reverse(A, A + cnt);
    return dfs(0, 0, true);
}

void solve()
{
	int a,b;
	while(cin >> a >> b >> mod)
		cout << f(b) - f(a-1) << endl;

}
```

**注意：**
```cpp
	if(limit)		//被限制，或者也没有前导0（限制），返回值即可
	{
		return ans;
	}
```
必须是被限制的不更新，没被限制的是可以记忆化更新的，所以是limit，不是!limit

这里逻辑如果不小心写错了，记忆化搜索的剪枝就会失效，虽然能够跑出来的答案是正确的，但是时间复杂度已经退化为阶乘级别！！！
