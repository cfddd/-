## 题目
- [C. Binary Search](https://codeforces.com/problemset/problem/1436/C)
## 题意
![](https://img2023.cnblogs.com/blog/2740326/202304/2740326-20230405204340314-630825628.png)
- 给一个数字n，构造出一个全排列的数组a，满足上面二分结果为true
- 请求出不同全排列数组a的数量，答案模1e9+7
## 思路
- 模拟：按照二叉查找树的思路，模拟这个二分所有可能遇到的mid，使得判断条件成立（为什么落在最后的点上？因为是折半查找，搜索树上没有重复的节点）
- 这样得到了必须大于**等于**x位置的数量，和必须小于x位置的数量，也就是知道了剩下没有遇到的位置该如何排列（全排列）
- `ans = C[Cles][les.size()] * fact[les.size()] % mod;`计算小于的数量（特殊处理遇到pos节点的位置，即忽略）
  `ans = (ans * (C[Cgre][gre.size()] * fact[gre.size()] % mod)) % mod;`计算大于的数量
  `ans = (ans * fact[n - les.size() - gre.size() - 1]) % mod;`计算剩余的排列数量
- 需要用到组合数和计数原理，和二分
## 代码
```
const double PI = acos(-1.0);
const int N = 1010, mod = 1e9 + 7;
int C[N][N];
int fact[N];
void solve()
{
    int n, x, pos;
    cin >> n >> x >> pos;
    vector<int> gre, les;
    vector<int> idx;
    int l = 0, r = n;
    while(l < r)
    {
        int mid = l + r >> 1;
        idx.push_back(mid);
        if(mid <= pos)
        {
            l = mid + 1;
            if(mid != pos)les.push_back(mid);
        }
        else
        {
            r = mid;
            gre.push_back(mid);
        }
        // debug1(mid);
    }
    
    int Cles = x - 1;
    int Cgre = n - x;
    // debug2(Cles, Cgre);
    // debug2(les.size(), gre.size());
    int ans = C[Cles][les.size()] * fact[les.size()] % mod;
    ans = (ans * (C[Cgre][gre.size()] * fact[gre.size()] % mod)) % mod;
    ans = (ans * fact[n - les.size() - gre.size() - 1]) % mod;
    cout << ans << endl;
}

signed main()
{

    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);

    C[0][0] = fact[0] = 1;
    for (int i = 1; i <= 1000;i ++)
    {
        C[i][0] = C[i][i] = 1;
        fact[i] = (fact[i - 1] * i) % mod;
        for (int j = 1; j < i;j ++)
        {
            C[i][j] = (C[i - 1][j] + C[i - 1][j - 1]) % mod;
        }
    }

        // caseT
        solve();

    return 0;
}
```