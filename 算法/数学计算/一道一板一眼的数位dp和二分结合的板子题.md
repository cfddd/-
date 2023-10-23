## 题目
[1811E - Living Sequence](https://codeforces.com/problemset/problem/1811/E)
## 题意
- 找出第n个，数位中不含‘4’的数字
## 思路
- 数位dp + 二分
- 唯一要注意的就是纯dfs搜索会卡常（hh，就是复杂度太高了），用上一点记忆化
## 代码
```
const int N = 14;
int dp[N][N];
int a[N];
int len;
 
void ini()
{
    for (int i = 0; i <= 9; i++)
        dp[0][i] = 1;
    dp[0][4] = 0;
 
    for (int i = 1; i < N; i++) // 从低位到高位
    {
        for (int j = 0; j <= 9; j++)
        {
            if (j == 4)
                continue;
            for (int k = 0; k <= 9; k++)
            {
                if (k == 4)
                    continue;
                dp[i][j] += dp[i - 1][k];
            }
        }
    }
}
 
int cal(int x)
{
 
    if (!x)
        return 1;
    len = 0;
    while (x)
        a[len++] = x % 10, x /= 10;
 
    int last = -1;
    int res = 0;
    for (int i = len - 1; i >= 0; i--)
    {
        for (int j = 0; j < a[i]; j++)
        {
            if (j == 4)
                continue;
            res += dp[i][j];
        }
 
        if (a[i] == 4)
            return res;
        last = a[i];
        if (!i)
            res++; // 最后一位的那个数字
    }
 
    return res;
}
 
void solve()
{
    int k;
    cin >> k;
    int l = 0, r = 4e12;
    while(l < r)
    {
        int mid = l + r + 1 >> 1;
        if(cal(mid) <= k)
            l = mid;
        else
            r = mid - 1;
    }
    cout << l + 1 << endl;
}
 
```