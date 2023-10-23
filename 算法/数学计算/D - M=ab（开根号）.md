## 题意
- 给两个整数n，m（1 ~ 1e12）
- 找到两个1 ~ n之内的数字，它的大于等于m并且最小
## 思路
![](https://img2023.cnblogs.com/blog/2740326/202304/2740326-20230402151304133-1341384930.jpg)
## 代码
```
void solve()
{
    int n, m;
    cin >> n >> m;

    int ans = 9e18;
    int limit = sqrtl(m);
    while((limit - 1)*(limit - 1) >= m)
        limit--;
    while ((limit + 1) * (limit + 1) < m)
        limit++;
    for (int i = 1; i <= limit+1; i++)
    {
        int t = (m + i - 1) / i;
        // debug2(t,i);
        if(t <= n && i <= n)
            ans = min(ans, t * i);
    }
    if(ans == 9e18)
        cout << -1 << endl;
    else printf("%lld", ans);
}
```