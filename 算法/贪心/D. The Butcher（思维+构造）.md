## 题目
[Codeforces Round 866 (Div. 2)D. The Butcher](https://codeforces.com/contest/1820/problem/D)
## 题意
- n个数对a，b，表示矩形
- 这n个矩形通过原先一个大矩形分割而来
- 每次分割只在上一次分割的矩阵其中之一
- 现在原先的矩阵大小未知，问有原先的矩阵（在切割过程中不会旋转矩阵）多少种，并输出
- 保证至少存在一种方法

## 思路
- 从题意可得，在所有子矩阵中，必然有一条边和原先矩阵完全重合且相等，这条边最大
- 所以可以通过这一条最大的边，来判断是否能把矩阵拼好
- 同时，我们还能得出原先矩阵的面积，通过这一条最长边，可以算出另一条对应的长度
- 通过一个分解函数，来逐步分割检查能不能拼成，具体思路见代码
## 代码
```cpp
void solve()
{
    int n;
    cin >> n;
    map<int, multiset<int>> a, b;
    set<PII> ans;
    int sum = 0;
    int maxa = 0, maxb = 0;
    for (int i = 0; i < n;i ++)
    {
        int x,y;
        cin >> x >> y;
        a[x].insert(y);
        b[y].insert(x);
        sum += x * y;
        maxa = max(maxa, x);
        maxb = max(maxb, y);
    }

    auto check = [&](int X, int Y, int id, map<int, multiset<int>> xx, map<int, multiset<int>> yy)
    {
        int start = n;
        while(X > 0 && Y > 0)
        {
            
            int temp = start;
            //X
            if(id == 0)
            {
                for(auto it:xx[X])
                {
                    Y -= it;
                    // debug2(Y, it);
                    // yy.erase(it);//删除所有yy中为it的元素，应该是只删除一个
                    yy[it].erase(yy[it].find(X));
                    temp--;
                }
                xx.erase(X);
                id ^= 1;
            }
            else    //Y
            {
                // debug3(X, Y,yy.count(Y));
                for (auto it : yy[Y])
                {
                    X -= it;
                    // debug2(X, it);
                    xx[it].erase(xx[it].find(Y));
                    temp--;
                }
                yy.erase(Y);
                id ^= 1;
            }
            
            if(temp == start)
                return false;
            start = temp;
            
        }
        return X == 0 || Y == 0;
    };

    if ((sum % maxa) == 0 && check(maxa, sum / maxa, 0,a,b))
        ans.insert({maxa, sum / maxa});
    if ((sum % maxb) == 0 && check(sum / maxb, maxb, 1,a,b))
        ans.insert({sum / maxb, maxb});

    cout << ans.size() << endl;
    for(auto x:ans)
        cout << x.fi << " " << x.se << endl;
}
```
