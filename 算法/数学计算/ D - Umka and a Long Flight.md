## 题目
[ D - Umka and a Long Flight](https://codeforces.com/problemset/problem/1811/D)
## 题意
- 给一个整数n（1 <= n && n <= 44）
- 再给衣柜坐标x，y，从1开始，表示在一个长为f(n+1)，宽为f(n)的长方形里的某一个1*1的小正方形
- 长为f(n+1)，宽为f(n)的长方形，由两个1*1的正方形拼成，然后由2 ~ n边长的正方形拼成
- f(n)表示斐波那契数列，f[0] = f[1] = 1
- 把x，y这个小正方形染色，问这个正方形能不能是最小正方形之一
## 思路
- 用上下左右四个边界把这个长方形固定在坐标轴里
- 从n开始模拟，每次剪掉一个n*n的正方形，更新正方形的划定位置，可以发现每次只有两种方法
- 判断这个x，y能不能在留下来的长方形里面，直到n = 1，如果能，那么这个答案就是对的
- 如果中途失败，可以肯定不能
## 代码
```
int n, x, y;
const int N = 50;
int f[N];
void solve()
{
    cin >> n >> x >> y;
    int up = 1, down = f[n];
    int l = 1, r = f[n+1];
    while(n > 1)
    {
        // debug4(up, down, l, r);
        if (down - up + 1 > r - l + 1)
        {
            if (down - f[n] >= x && x >= up)
            {
                down -= f[n];
            }
            else if (up + f[n] <= x && x <= down)
            {
                up += f[n];
            }
            else
            {
                cout << "NO" << endl;
                return;
            }
        }
        else
        {

            if (r - f[n] >= y && y >= l)
            {
                r -= f[n];
            }
            else if (l + f[n] <= y && y <= r)
            {
                l += f[n];
            }
            else
            {
                cout << "NO" << endl;
                return;
            }
        }
        n--;
    }
    
    cout << "YES" << endl;
}

signed main()
{

    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);

    f[0] = f[1] = 1;
    for (int i = 2; i <= 45;i ++)
        f[i] = f[i - 1] + f[i - 2];

    caseT
    solve();

    return 0;
}
/*

*/
```