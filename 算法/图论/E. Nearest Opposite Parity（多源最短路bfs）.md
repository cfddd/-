## 题目
-  [Nearest Opposite Parity（多源最短路bfs）](https://codeforces.com/problemset/problem/1272/E)
## 题意
![](https://img2023.cnblogs.com/blog/2740326/202302/2740326-20230219154346694-1401716333.png)
## 思路
- 多源最短路
## 代码
```

const int N = 2e5+10;
int a[N];
vector<int> edge[N];
int dist[N];
int ans[N];
void bfs(vector<int> &st,vector<int> &ed)
{
    queue<int> q;
    memset(dist,0x3f,sizeof dist);
    for(auto x:st)
    {
        q.push(x);
        dist[x] = 0;
    }

    while(q.size())
    {
        int u = q.front();q.pop();
        for(auto v:edge[u])
        {
            if(dist[v] != INF)continue;
            dist[v] = dist[u] + 1;
            q.push(v);
        }
    }

    for(auto x:ed)
    {
        if(dist[x] == INF)ans[x] = -1;
        else ans[x] = dist[x];
    }
}

void solve() 
{
    int n;cin >> n;
    vector<int> odd,even;
    
    for(int i = 1;i <= n;i ++)
    {
        cin >> a[i];
        int l = i - a[i],r = i + a[i];
        if(l >= 1)edge[l].push_back(i);
        if(r <= n)edge[r].push_back(i);
        if(a[i]&1)odd.push_back(i);
        else even.push_back(i);
    }

    bfs(odd,even);
    bfs(even,odd);

    for(int i = 1;i <= n;i ++)cout << ans[i] << " ";

}
signed main()
{

    // ios::sync_with_stdio(false);
    // cin.tie(0);
    // cout.tie(0);

    // caseT
    solve();
    
    return 0;

}
/*

*/
```