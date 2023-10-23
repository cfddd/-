## 题目
- [B. Greg and Graph](https://codeforces.com/contest/295/problem/B)
## 题意

- 输入 n(1≤n≤500) 表示 n 个点的有向完全图，然后输入 n*n 的邻接矩阵 a，其中 a[i][j] 表示 i 到 j 的边权，范围 [1,1e5]（特例是 a[i][i]=0）。
- 图的节点编号从 1 开始。
- 然后输入 1~n 的排列，表示我要一个个地删除图上的点，每删除一个点，这个点的出边和入边都会被删除。
- 输出 n 个数，第 i 个数表示第 i 次删除之前，所有剩余点对的最短路之和。

## 思路
- Floyd
- Floyd算法差点可以倒着a的顺序插入，这样节省了一次建图
## 代码
```cpp
const int N = 510;
int f[N][N];
int n;
int a[N],ans[N];

void floyd(int tar,int ed)
{
    for (int u = 0; u < n;u ++)
    {
        for (int v = 0; v < n;v ++)
        {
            f[a[u]][a[v]] = min(f[a[u]][a[v]], f[a[u]][tar] + f[tar][a[v]]);
        }
    }
}

void solve()
{
    cin >> n;
    for (int i = 0; i < n;i ++)
        for (int j = 0; j < n;j ++)
            cin >> f[i][j];

    for (int i = 0; i < n;i ++)
    {
        cin >> a[i];
        a[i]--;
    }
    reverse(a, a + n);

    for (int i = 0; i < n;i ++)
    {
        int add = a[i];

        floyd(add, i);

        int res = 0;
        for (int u = 0; u <= i;u ++)
        {
            for (int v = 0; v <= i;v ++)if(u != v)
            {
                res += f[a[u]][a[v]];
                // debug3(u, v, f[u][v]);
            }
        }
        ans[n - i - 1] = res;
    }

    for (int i = 0; i < n;i ++)
        cout << ans[i] << " ";
}
```