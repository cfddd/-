## 题目

Codeforces Round 142 (Div. 1) B - Planets<https://codeforces.com/contest/229/problem/B>

## 题意

<https://codeforces.com/problemset/problem/1139/C>

输入 n(2≤n≤1e5) k(2≤k≤100) 和一棵无向树的 n-1 条边（节点编号从 1 开始），每条边包含 3 个数 x y c，表示有一条颜色为 c 的边连接 x 和 y，其中 c 等于 0 或 1。

对于长为 k 节点序列 a，走最短路，按顺序经过节点 a1 -> a2 -> ... -> ak。
对于所有长为 k 的节点序列 a（这有 n^k 个），统计至少经过一条 c=1 的边的序列 a 的个数。

## 思路

- dijkstra的基础上稍微改进
- 对加上时间限制后可能的影响讨论，首先需要假设dist[u]及相关之前的点已经更新为最近距离
  - 如果到达点v的距离dist[u] + len在t[v]集合之外，无影响
  - 如果到达点v的距离dist[u] + len在t[v]集合之内，那么下次出队时，必然要更新dist[v]为最近的不包含的较大点
  - 以为dijkstra每次必然是最近的点（没有比自己更近的点）出队更新其他点，，其他点即使不受影响（不延后）也无法更新这个点，所以用dij算法的正确性是可以保障的
  - 这样的更新操作可以是dijkstra的子集（延后某些dist，使得dist始终都在变大，但是算法执行时的非递减属性没有改变）
- 所以我们在dij出队后，更新dist即可
- 可以使用二分或者hash表记录下一个合法位置的出现

## 代码

```cpp
const int N = 1e5 + 10, M = 2 * N;
int h[N], e[M], ne[M], w[M], idx;
int dist[N];
bool st[N];

void add(int a, int b, int c)
{
    e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx++;
}

void solve()
{
    int n, m;
    cin >> n >> m;
    memset(h, -1, sizeof h);
    memset(dist, 0x3f, sizeof dist);
    for (int i = 1; i <= m; i++)
    {
        int u, v, w;
        cin >> u >> v >> w;
        add(u, v, w);
        add(v, u, w);
    }

    map<int, map<int, int>> nex;
    for (int i = 1; i <= n; i++)
    {
        int cnt;
        cin >> cnt;
        vector<int> t(cnt);
        for (auto &x : t)
            cin >> x;
        for (int j = cnt - 1; j >= 0; j--)
        {
            int v = t[j];
            if (nex[i].count(v + 1))
                nex[i][v] = nex[i][v + 1];
            else
                nex[i][v] = v + 1;
        }
        // for(auto x:t)debug2(x,nex[i][x]);
    }

    dist[1] = 0;
    priority_queue<PII, vector<PII>, greater<PII>> pq;
    pq.push({0, 1});
    while (pq.size())
    {
        int u = pq.top().se;
        pq.pop();

        if (u == n)
            break; // sb

        if (st[u])
            continue;
        st[u] = 1;
        if (nex.count(u) && nex[u].count(dist[u]))
            dist[u] = nex[u][dist[u]];
        // debug2(u,dist[u]);

        for (int i = h[u]; i != -1; i = ne[i])
        {
            int v = e[i];
            if (dist[v] > dist[u] + w[i])
            {
                dist[v] = dist[u] + w[i];
                pq.push({dist[v], v});
            }
        }
    }

    if (dist[n] == 0x3f3f3f3f) cout << -1 << endl;
    else cout << dist[n] << endl;
}
```

## debug过程

- 一开始wa3偷看样例发现，到达最后一个点之后不需要再更新，直接结束dij即可
- 然后一直wa，最后发现是map的count位置写错了，相当隐蔽，很难发现！！！
- 解决办法？把不同逻辑部分分成块！然后逐块检查！要求不漏掉任何一个逻辑，检查所有的变量是否符合自己的预期（交给队友一起来）
- G，一直以为是dij的问题，结果是bug
- 狗狗代码
```cpp
const int N = 1e5+10,M = 2*N;
int h[N],e[M],ne[M],w[M],idx;
int dist[N];
bool st[N];
void add(int a,int b,int c)
{
    e[idx] = b,w[idx] = c,ne[idx] = h[a],h[a] = idx ++;
}
 
void solve()
{
    int n,m;cin >> n >> m;
    memset(h,-1,sizeof h);
    memset(dist,0x3f,sizeof dist);
    for(int i = 1;i <= m;i ++)
    {
        int u,v,w;cin >> u >> v >> w;
        add(u,v,w);
        add(v,u,w);
    }
 
    map<int,map<int,int>> nex;
    for(int i = 1;i <= n;i ++)
    {
        int cnt;cin >> cnt;
        vector<int> t(cnt);
        for(auto &x:t)cin >> x;
        for(int j = cnt-1;j >= 0;j --)
        {
            int v = t[j];
            if(nex.count(v+1))nex[i][v] = nex[i][v+1];//就是这个count，debug的过程漏了，只有一条，一直以为是dij的问题，没有更新好
            else nex[i][v] = v+1;
        }
        // for(auto x:t)debug2(x,nex[i][x]);
    }
 
    dist[1] = 0;
    priority_queue<PII,vector<PII>,greater<PII>> pq;
    pq.push({0,1});
    while(pq.size())
    {
        int u = pq.top().se;
        pq.pop();
 
        if(u == n)break;//sb
 
        if(st[u])continue;
        st[u] = 1;
        if(nex.count(u) && nex[u].count(dist[u]))dist[u] = nex[u][dist[u]];
        // debug2(u,dist[u]);
 
        for (int i = h[u]; i != -1; i = ne[i])
        {
            int v = e[i];
            if (dist[v] > dist[u] + w[i])
            {
                dist[v] = dist[u] + w[i];
                pq.push({dist[v], v});
            }
        }
    }
 
    if(dist[n] == 0x3f3f3f3f)
    {
        cout << -1 << endl;
    }else cout << dist[n] << endl;
}
```
