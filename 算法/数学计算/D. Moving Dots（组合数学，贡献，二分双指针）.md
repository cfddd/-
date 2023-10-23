## 题目
- https://codeforces.com/contest/1788/problem/D
## 题意
- 给一个长度为n的x数组，代表n个点的坐标
- 每个点会向离他最近的点移动，当有点相遇时，停止
- 问任意子序列（点数量大于等于2），有多少个终点
## 思路
- 从题目给的“2”这个信息入手，从贡献这个方面来考虑
- 对于任意两不同的点，具有一定的范围，让这个范围内的点都被吸过来
- 于是范围外的点能保证两点相互靠近产生一次贡献，所以，这些范围外的数字可选可不选
- 对于（i,j）
  - l = lower_bound(x+1,x+n+1,x[i] - len)
  - r = lower_bound(x+1,x+n+1,x[j] + len)
  - 贡献为 $2^{l + n - r + 1}$
##代码
```
const int N = 3010;
const int mod = 1e9+7;
int x[N];
int qmi(int x,int y)
{
    int res = 1;
    int p = x;
    while(y)
    {
        if(y & 1)res = (res * p) % mod;
        y >>= 1;
        p = (p * p) % mod;
    }
    return res;
}
void solve() 
{
    int n;cin >> n;
    for(int i = 1;i <= n;i ++)
        cin >> x[i];

    int ans = 0;
    for(int i = 1;i <= n;i ++)
    {
        for(int j = i+1;j <= n;j ++)
        {
            int len = x[j] - x[i];
            int l = lower_bound(x+1,x+n+1,x[i] - len) - x - 1;
            int r = lower_bound(x+1,x+n+1,x[j] + len) - x;
            // debug2(l,r);
            ans = (ans + qmi(2,l + n - r + 1)) % mod;
            // debug1(ans);
        }
    }

    cout << ans << endl;
}
signed main()
{

    // ios::sync_with_stdio(false);
    // cin.tie(0);
    // cout.tie(0);

    // caseT
    solve();
    
    return 0;

}

```