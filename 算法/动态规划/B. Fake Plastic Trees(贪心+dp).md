## 题目

(Fake Plastic Trees)[https://codeforces.com/problemset/problem/1693/B]

## 题意
输入 T(≤1e3) 表示 T 组数据。所有数据的 n 之和 ≤2e5。
每组数据输入 n(2≤n≤2e5) 表示一棵 n 个节点的树，编号从 1 开始，1 为根节点。
然后输入 p[2],p[3],...,p[n]，其中 p[i] 表示 i 的父节点。
然后输入 n 行，其中第 i 行输入两个数 l 和 r，表示第 i 个节点值的目标范围 [l,r]。

初始时，所有节点值均为 0。
每次操作你可以选择一条从 1 开始的路径，把路径上的每个节点值都加上一个数。要求这些数按照路径的顺序，形成一个递增序列。（可以相等，可以等于 0，例如 [0,0,1,3,3]）
要使所有节点值都在对应的范围内，至少要操作多少次？

## 思路

先满足叶子节点，这样肯定是最优的。

在满足叶子的情况下，因为是非递减序列，所以让序列尽量大也是最优的

设dp[u]为操作数最少的情况下，满足子树后，u节点能得到的最大权重。

$$ dp[u] = min(\sum_{u子节点v} dp[v],R[u]) $$

## 代码

```cpp
const int N = 2e5+10;

vector<int> p[N];
int L[N],R[N];
int ans = 0;
LL dfs(int u)
{
    LL sum = 0;
    for(auto v:p[u])
    {
        sum += dfs(v);
    }
    if (sum < L[u])sum = R[u],ans ++;
    return min(sum,1LL*R[u]);
}

void solve()
{
    ans = 0;
    int n;cin >> n;
    for(int i = 1;i <= n;i ++)p[i].clear();
    for(int i = 2;i <= n;i ++){
        int t;cin >> t;
        p[t].push_back(i);
    }
    for(int i = 1;i <= n;i ++)cin >> L[i] >> R[i];
    dfs(1);
    cout << ans << endl;

}


```
