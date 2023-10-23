## 题目
- https://codeforces.com/contest/1790/problem/F
## 题意
- 给一颗根节点为$c_0$树，有n个节点，n-1条边
- 根节点已经被染成了黑色，其他所有节点都是白色
- 每次染黑一个节点$c_i$，问所有已经染黑的节点的距离最短是多少
## 思路
- 维护一个dist数组，dist[i]表示节点 i 到黑色节点的最短距离（如果 i 已经是黑色节点，为0）
- 从$c_i$开始搜索，把$c_i$点染黑后，bfs更新搜索到的点的dist，并入队
- 用下一次染黑的节点dist更新ans，并输出答案
- 时间复杂度$O(N\sqrt N)$
## 代码
```
#include<bits/stdc++.h>
using namespace std;
#define CaseT int CaseT; cin >> CaseT; while(CaseT--)
 
const int N = 2e5 + 10;
int c[N]; 
int dist[N]; 
 
void solve() {
    int n; cin >> n;
    vector<vector<int>> g(n + 1);
 
    for (int i = 1; i <= n; i++) {
        cin >> c[i];
    }
 
    int m = n - 1;
    while (m--) {
        int u, v;
        cin >> u >> v;
        g[u].push_back(v), g[v].push_back(u);
    }
 
    for (int i = 1; i <= n; i++) {
        dist[i] = n;
    }
 
    int ans = n;
    for (int i = 1; i <= n; i++) {
        ans = min(ans, dist[c[i]]);
 
        if (i > 1) {
            ans = min(ans, dist[c[i]]);
            cout << ans << " ";
        }
 
        queue<int> q;
        q.push(c[i]);
        dist[c[i]] = 0;
 
        while (q.size()) {
            int u = q.front(); q.pop();
            
            for (auto v : g[u]) {
                if (dist[u] + 1 < min(dist[v], ans)) {
                    dist[v] = dist[u] + 1;
                    q.push(v);
                }
            }
        }
    }
    cout << endl;
}
 
int main() {
    CaseT
    solve();
}
```