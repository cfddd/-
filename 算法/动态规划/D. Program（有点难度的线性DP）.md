## 题目

[D. Program](https://codeforces.com/problemset/problem/1473/D)

## 题意

给一个长度为n的‘+’，‘-’序列，表示+1和-1
在给m个查询，问忽略[l,r]之间的序列，能走到多少个不同的数字

## 思路

- 分为前后缀计算，前缀计算比较简单关键是后缀计算
- 后缀上，需要关注能够到达的最小值和最大值
- 定义sufL[i]和sufR[i]分别表示为到达的最小值和最大值
- 可以得出转移方程
  - now = s[i] == '+' ? 1 : -1;
  - sufR[i] = max(sufR[i + 1] + now, 0);
  - sufL[i] = min(sufL[i + 1] + now, 0);

## 代码

```cpp
const int N = 2e5+10;
char s[N];
int preL[N], preR[N], pre_cur[N];
int sufL[N], sufR[N];
void solve()
{
    int n, m;
    cin >> n >> m;
    cin >> s + 1;
    preL[0] = preR[0] = pre_cur[0] = 0;
    for (int i = 1; i <= n;i ++)
    {
        pre_cur[i] = pre_cur[i - 1] + (s[i] == '+' ? 1 : -1);
        preL[i] = min(pre_cur[i], preL[i - 1]);
        preR[i] = max(pre_cur[i], preR[i - 1]);
    }

    sufL[n + 1] = sufR[n + 1] = 0;
    for (int i = n; i >= 1;i --)
    {
        int now = s[i] == '+' ? 1 : -1;
        sufR[i] = max(sufR[i + 1] + now, 0);
        sufL[i] = min(sufL[i + 1] + now, 0);
        // debug2(sufL[i], sufR[i]);
    }

    for (int i = 1; i <= m; i++)
    {
        int l, r;
        cin >> l >> r;
        l--;r++;
        int maxx = max(0, preR[l]), minn = min(0, preL[l]);
        maxx = max(maxx, sufR[r] + pre_cur[l]);
        minn = min(minn, sufL[r] + pre_cur[l]);
        // debug2(maxx, minn);
        cout << maxx - minn + 1 << endl;
    }
}
```
