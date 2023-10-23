## D. Directed Roads
![img](https://img2023.cnblogs.com/blog/2740326/202305/2740326-20230526234840990-1349123322.png)
![img](https://img2023.cnblogs.com/blog/2740326/202305/2740326-20230526234901441-2090622680.png)
## 思路
- 环外的边可以随意变换，每个环上的的贡献是$2^{环的大小} - 2$，相乘就是答案
- 在写代码发现构建了无向图，有点问题，记录一下
![img](https://img2023.cnblogs.com/blog/2740326/202305/2740326-20230526234840990-1349123322.png)

可以发现，做完拓扑有很多点的度数是负数，才发现无向图的拓扑排序后，度数 < 1 的点都是环外的点

总之，很有必要记录一下无向图的拓扑排序
## 代码
```cpp

const int N = 2e5 + 10, mod = 1e9 + 7;
vector<int> edge[N];
int d[N];
int fa[N];
int siz[N];

int find(int u)
{
    if (fa[u] != u)
        fa[u] = find(fa[u]);
    return fa[u];
}

int qmi(int x, int y)
{
    int res = 1;
    while (y)
    {
        if (y & 1)
            res = (1LL * res * x) % mod;
        x = (1LL * x * x) % mod;
        y >>= 1;
    }
    return res;
}

void solve()
{
    int n;
    cin >> n;

    for (int i = 1; i <= n; i++)
        fa[i] = i, siz[i] = 1;

    for (int i = 1; i <= n; i++)
    {
        int u;cin >> u;
        edge[u].push_back(i);edge[i].push_back(u);
        d[u]++;d[i]++;
    }

    int cir_out = 0;

    queue<int> q;
    for (int i = 1; i <= n; i++)if (d[i] == 1)q.push(i);
    while (q.size())
    {
        int u = q.front();q.pop();
        cir_out++;
        for (auto v : edge[u])
        {
            d[u]--;d[v]--;
            if (d[v] == 1)
                q.push(v);
        }
    }

    for (int u = 1; u <= n; u++)
    {
        if (d[u] < 1)continue;
        for (auto v : edge[u])
        {
            if (d[v] < 1)continue;
            int pu = find(u), pv = find(v);
            if (pu != pv)
            {
                siz[pv] += siz[pu];
                fa[pu] = pv;
            }
        }
    }

    LL ans = qmi(2, cir_out);
    for (int u = 1; u <= n; u++)
    {
        if (find(u) == u && siz[u] > 1)
        {
            ans = (ans * ((qmi(2, siz[u]) - 2 + mod) % mod)) % mod;
        }
    }
    cout << ans << endl;

}

```