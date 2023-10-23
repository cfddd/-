## 题目
- https://atcoder.jp/contests/arc111/tasks/arc111_b
## 题意
- 输入 n(≤2e5) 和一个 n 行 2 列的矩阵，矩阵元素范围 [1,4e5]
- 从每行中恰好选一个数，最多能选出多少个不同的数
## 思路
- 从图的方向去思考
- 建图，发现环上的点所有都可以取到
- 分支上的点，可以通过在拓扑排序的时候，计算出边得数量（即环外点的数量），这些点也是可以取到的
- 但是，如果一个连通块是无环的（是一棵树），那么这颗树的顶点不被取到
- 于是做法分两步
  - 拓扑找环，同时把度数不为0的点插入（恰好把一棵树的顶点忽略）答案集合
  - 环上所有点（度数大于1），插入答案集合
## 代码
```
const int N = 2e5+10,M = 2*N;
vector<int> edge[M];
int d[M];
void solve() 
{
    int n;cin >> n;
    for(int i = 0;i < n;i ++)
    {
        int a,b;cin >> a >> b;
        edge[a].push_back(b);
        edge[b].push_back(a);
        d[a] ++;
        d[b] ++;
    }

    set<int> ans ;
    queue<int> q;
    for(int i = 1;i <= 400000;i ++)if(d[i] == 1)q.push(i);
    while(q.size())
    {
        int u = q.front();q.pop();
        for(auto v:edge[u])
        {
            d[v] --;
            if(d[v] == 1)q.push(v);
            if(d[u] > 0)ans.insert(u);
        }
    }
    for(int i = 1;i <= 400000;i ++)if(d[i] > 1)ans.insert(i);

    cout << ans.size() << endl;

}
```