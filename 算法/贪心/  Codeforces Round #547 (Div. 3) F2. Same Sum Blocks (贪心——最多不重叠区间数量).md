## 题目
- https://codeforces.com/problemset/problem/1141/F2
## 题意
- 忽略；
- 给出一个数组，求和相等的，不重叠子串的最大数量，并输出
- （题目有点绕）
## 思路
- 先求出数组前缀和，然后找出所有数字和的区间数组
- 对不同的和进行贪心操作————找最多不重叠区间数量
- 一个经典思路，按照右边界排序（左端点升序），遍历直到当前区间左边界大于上一次区间的右边界，更新左边界，然后计数
## 代码
```

//常数定义
const double eps = 1e-6;
const double PI = acos(-1.0);
const int INF = 0x3f3f3f3f;
const int N = 1510;

int a[N];
void solve() 
{
    int n;cin >> n;
    for(int i = 1;i <= n;i ++)
    {
        cin >> a[i];
        a[i] += a[i-1];
    }

    map<int,vector<PII>> seg;
    for(int i = 1;i <= n;i ++)
    {
        for(int j = 1;j <= i;j ++)
        {
            seg[a[i] - a[j-1]].push_back({j,i});
        }
    }

    vector<PII> ans;
    for(auto &[id,edge]:seg)
    {
        vector<PII> res;
        int r = -INF;
        for(auto x:edge)
        {
            if(x.fi > r)r = x.se,res.push_back(x);
        }
        if(ans.size() < res.size())ans = res;
    }

    cout << ans.size() << endl;
    for(auto [l,r]:ans)cout << l << " " << r << endl;
}


```
