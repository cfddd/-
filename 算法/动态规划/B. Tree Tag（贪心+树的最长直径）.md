## 题目

[B. Tree Tag](https://codeforces.com/problemset/problem/1404/B)

## 题意

![img](https://img2023.cnblogs.com/blog/2740326/202304/2740326-20230420210130124-971151525.png)

## 思路

- 因为这是一颗树，所以不管怎么追逐，我们都可以理解为在同一条路上追逐（去掉我们不走的路，就是一个线段）
- 首先，如果da > db，显然能追上，进一步，da == db时，因为路径的长度是有限的，也显然可以追上
- 因为树上任意两点的最短路径是固定的，所以a点可以一直朝着b追，而b是无法走回头路的（至少在a的范围外）
- 所以只存在当a刚好可以追上b时（da == dista_b）,b >2*a才可以逃脱，bob胜利
- 还要注意一点，如果整棵树的最长直径不大于2*a，那么显然bob无法逃脱，alice胜利。对于这一步，可以**跑树的最长直径板子**
- 还有一点，alice先手，直接抓到bob，alice胜利。**判断ab之间距离**

## 代码

```cpp
const int N = 2e5+10;
int h[N], e[N], ne[N], idx;
void add(int a,int b)
{
    e[idx] = b;
    ne[idx] = h[a];
    h[a] = idx++;
}

int maxlen = 0, distab = 0;
int n, a, b, da, db;
int dfs(int u,int fa)
{
    int l0 = 0, l1 = 0;
    for (int i = h[u]; i != -1;i = ne[i])
    {
        if(e[i] == fa)
            continue;
        int len = dfs(e[i], u) + 1;
        if(len >= l0)
            l1 = l0, l0 = len;
        else if(len > l1)
            l1 = len;
    }
    maxlen = max(maxlen, l0 + l1);
    return l0;
}
int dfs1(int u, int fa, int dis)
{
    if(u == b)
        distab = dis;
    for (int i = h[u]; i != -1; i = ne[i])
    {
        if (e[i] == fa)
            continue;
        dfs1(e[i], u, dis + 1);
    }
}
void solve()
{
    maxlen = 0;
    cin >> n >> a >> b >> da >> db;
    for (int i = 1; i <= n;i ++)
        h[i] = -1;
    for (int i = 0; i < n-1;i ++)
    {
        int a, b;
        cin >> a >> b;
        add(a, b);
        add(b, a);
    }

    if(da * 2 >= db)
    {
        cout << "Alice" << endl;
        return;
    }

    dfs(1, -1);
    // debug1(maxlen);
    dfs1(a, -1,0);
    if(maxlen <= da*2 || distab <= da)
        cout << "Alice" << endl;
    else
        cout << "Bob" << endl;
}

```
