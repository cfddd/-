## 题目
- [https://codeforces.com/problemset/problem/1361/B](https://codeforces.com/problemset/problem/1361/B)
## 题意
- 输入 t(≤1e5) 表示 t 组数据，每组数据输入 n(≤1e6) p(1≤p≤1e6) 和长为 n 的数组 k(0≤k[i]≤1e6)。所有数据的 n 之和不超过 1e6。

- 从这 n 个数中选出若干个数（可以为空）组成一组，剩余的数组成另一组。
- 每组计算 pow(p,k[i]) 之和。
- 输出这两个和的差值的最小值，对结果模 1e9+7。
## 思路
- 分组计算抽象为ans的加减
- 对k数组降序排序
- 首先，加上最大的 pow(p,k[i]) ，然后减去接下来的项，贪心把结果变小，直到 ans 减为0，只要数足够多，必然会出现这个过程（因为这是一个同底数的幂，可以当作一个p进制数，位数高的数肯定是能被单调递减的位数低的数恰好减掉的）
- 当上一个步骤减为0，那么回到最初是的贪心状态，加上一个最大的，为了缩小答案，逐渐减去后面的项，类似数学归纳
## 代码1
```
const double eps = 1e-6;
const double PI = acos(-1.0);
const int INF = 0x3f3f3f3f;
const int N = 1e6+10;
const int mod1 = 1e9+7,mod2 = 1e9+8;
int k[N];
int n,p;

int qmi(int a,int b,int mod)//快速幂在于思想
{
    int res=1;
    while(b)//对b进行二进制化,从低位到高位
    {
        if(b&1) res = res *a % mod;
        b >>= 1;
        a = a * a % mod;
    }
    return res;
}

void solve() 
{
    cin >> n >> p;
    for(int i = 0;i < n;i ++)cin >> k[i];
    sort(k,k+n,greater<int>());

    int ans1 = 0,ans2 = 0;
    for(int i = 0;i < n;i ++)
    {
        if(!ans1 && !ans2)
        {
            ans1 = (ans1 + qmi(p,k[i],mod1)) % mod1;
            ans2 = (ans2 + qmi(p,k[i],mod2)) % mod2;
        }else {
            ans1 = (ans1 + mod1 - qmi(p,k[i],mod1)) % mod1;
            ans2 = (ans2 + mod2 - qmi(p,k[i],mod2)) % mod2;
        }
    }

    cout << ans1 << endl;

}

```

## 一点点思考
- trick：判断原来的数字是否真的在原始意义上减为0，可以用双模数的方法（评测范围在mod1*mod2）
- 这个题目的数字大小是指数级别的，双模数的范围明显��不够的（我用mod2 = 998244353在第44个样例wa了）
- 但是对于无限的卡掉这个位置的点，测试数据是有限的（所以我改成mod2 = 1e9+8过了）
- 所以这个方法并不太实用，但是应该不会很倒霉恰好被卡掉![](https://img2023.cnblogs.com/blog/2740326/202302/2740326-20230211182007256-1977649419.png)
## 正解————栈
- 基本贪心路线一致
- [https://codeforces.com/blog/entry/78355]()https://codeforces.com/blog/entry/78355
- 那最大的数字除掉后面的小的数字，必然是一个整数，然后一通操作
## 代码2
- [https://codeforces.com/blog/entry/78355]()https://codeforces.com/blog/entry/78355
## 无敌解————写一个超级详细的模数类，然后就可以按照第一种方法随便搞
- ：）
