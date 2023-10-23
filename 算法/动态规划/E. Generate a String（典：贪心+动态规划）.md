## 题目

[E. Generate a String](https://codeforces.com/problemset/problem/710/E)

## 题意

- 输入三个不同的整数 $n(1 \leq n \leq 10^7)，x，y(1 ≤ x, y ≤ 10^9)$。
- 从 0 开始，每次可以 + 1 ， - 1 ，代价是x，或者当前值 * 2，代价是y
- 问怎样才能到达n用最小的代价

## 思路

- 第一方法是暴力搜索，但是数据过大，不行
- 观察发现，如果从n出发，做所有操作的逆操作（*2变为/2），答案集合数量较少
- 当n为偶数时，才能除2
- 连续执行两次+1或者-1然后再/2，不如先/2然后再-1+1
- 当n为奇数时，只能+1或者-1
- 用线性dp异常简洁
- 定义f[i]表示到达i的最小代价
- f[i] = min(f[i-1] + x,f[(i+1)/2] + y + (i&1)*x);

## 代码

```cpp
const int N = 2e7 + 10;
int f[N];

void solve()
{
    int n, x, y;
    cin >> n >> x >> y;
    for (int i = 1; i <= n;i ++)
    {
        f[i] = min(f[i - 1] + x, f[(i + 1) / 2] + y + (i & 1) * x);
    }
    cout << f[n] << endl;
}
```
