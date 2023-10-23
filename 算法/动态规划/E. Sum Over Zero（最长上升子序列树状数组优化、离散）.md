
## 题目

- https://codeforces.com/contest/1788/problem/E

## 题意

- 给一个长度为n的数组a
- 定义一个子段为[x,y]，价值是长度（y - x + 1）
- 要选择一个子段，必须$0 \leq \sum_{i = x}^y a[i]$
- 问所有不相交子段的最大价值和是多少
  
## 思路

- 普通的最长上升子序列模型
  - 定义f[i]：前i个元素构成的最大价值
  - 转移：f[i] = max(f[i-1],f[j-1] + i - j + 1)
  - 设x数组的前缀和数组sum，则j必须满足sum[j] > sum[i]
- 这样操作显然会超时
- 一个经典的思路————把**f[x] - x**看成一个整体，i又是一个定值，所以我们就可以用树状数组等数据结构来实现找到需要转移的最大位置
- 时间复杂度O（NlogN）
- 树状数组的下标对应前缀和的大小，但是，因为前缀和可能非常大或者非常小，所以我们需要离散化，再这个离散后的抽象大小相关位置上用树状数组
  - ：）居然还可以用二分作为排序后离散化找位置的工具（学到了）
- 树状数组上维护一个最大值，因为  **- j**  的存在，初始可能为负数，需要初始化最小
- 查找：比当前前缀和相对位置小的维护的最大值
- 插入：上一个已经找到有的结果f[i - 1] - i + 1;
## 代码
```

const int N = 2e5+10;

int tr[N];
int a[N],sum[N],sumb[N];
int f[N];
int n;

int lowbit(int x)  // 返回末尾的1
{return x & -x;}

void add(int x,int w){
    for(int i = x;i < N;i += lowbit(i)){
        tr[i] = max(tr[i],w);
    }
}

int query(int u){
    int res = -INF;
    for(int i = u;i > 0;i -= lowbit(i)){
        res = max(tr[i],res);
    }
    return res;
}

void solve() 
{
    cin >> n;
    for(int i = 1;i <= n;i ++){
        cin >> a[i];
        sumb[i] = sum[i] = sum[i-1] + a[i];
        
    }
    for(int i = 1;i <= n;i ++)tr[i] = -INF;
    sort(sumb+1,sumb+n+1);
    for(int i = 1;i <= n;i ++)
    {   
        int id0 = lower_bound(sumb+1,sumb+n+1,sum[i-1]) - sumb;
        int id1 = lower_bound(sumb+1,sumb+n+1,sum[i]) - sumb;
        add(id0,f[i-1] - i + 1);
        f[i] = max(f[i-1],query(id1) + i);
    }

    cout << f[n] << endl;
}
signed main()
{
    solve();
    return 0;
}
/*

*/
```