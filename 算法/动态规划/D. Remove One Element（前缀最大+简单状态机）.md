## 题目
- [D. Remove One Element](https://codeforces.com/contest/1272/problem/D)
## 题意
- 输入 n(2≤n≤2e5) 和长为 n 的数组 a(1≤a[i]≤1e9)。
- 从 a 中去掉一个数（也可以不去掉）。
- 输出 a 的最长严格递增连续子数组的长度。
## 思路
- 一种方法是前缀最长和后缀最长，加起来。这种方法比较简单。
- 用状态机来写，定义f[i][0/1]分别表示前缀最大到第 i 这个位置上是否用过这唯一一次删除机会
- $f[i][0] = a[i] > a[i-1]?f[i-1][0] + 1:1$
- $f[i][1] = max( a[i] > a[i-1]?f[i-1][0] + 1:1,a[i] > a[i-2]?f[i-2][0] + 1:1 )$
- 初始状态
    - f[1][0] = f[1][1] = 1;
    - f[2][0] = (a[2] > a[1]) + 1;
    - f[2][1] = 1;
## 代码
```
const int N = 2e5 + 10;
int a[N];
int f[N][2];
void solve()
{
    int n;
    cin >> n;
    for (int i = 1; i <= n;i ++)
        cin >> a[i];

    f[1][0] = f[1][1] = 1;
    f[2][0] = (a[2] > a[1]) + 1;
    f[2][1] = 1;

    for (int i = 3; i <= n; i++)
    {
        f[i][0] = f[i][1] = 1;
        if(a[i] > a[i - 1])
        {
            f[i][0] = f[i - 1][0] + 1;
            f[i][1] = f[i - 1][1] + 1;
        }

        if (a[i] > a[i - 2])
            f[i][1] = max(f[i][1], f[i - 2][0] + 1);
    }
    int ans = 1;
    for (int i = 1; i <= n;i ++)
        ans = max({ans, f[i][0], f[i][1]});
    cout << ans << endl;
}
```