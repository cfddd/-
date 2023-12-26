## 题目
- https://codeforces.com/contest/1808/problem/C
## 题意
- 给两个数 l 和 r  $( 1 ≤ l ≤ r ≤ 10^{18})$
- 请找出再这个范围内的一个数字，使得按数位这个数字中的数最大值和最小值之差最小
## 思路
- 当 l 和 r 的数位长度不一样时，可以输出和l长度相同的999...（贪心）
- 在此之外，数位dp的思想，进行搜索
- 定义$dfs(int st, int start, int ed, char maxLast, char minLast)$
- st 当前搜索到的数位
- start 左边界是否受限
- ed 右边界是否受限
- maxLast(mingLast) 当前搜索的最大和最小值 
## 代码
```
string a, b;
int ans = 10;
string res;
string temp;
void dfs(int st, int start, int ed, char maxLast, char minLast)
{
    if (maxLast - minLast >= ans)  //剪枝掉比当前不优的情况
        return;

    if (st == a.size())            //搜索结束
    {
        ans = maxLast - minLast;
        res = temp;
        return;
    }
    char l ;                       //搜索的起点和终点
    char r ;
    
    if (start && ed)
        l = a[st],r = b[st];
    else if (start)
        l = a[st],r = '9';
    else if (ed)
        l = '0',r = b[st];
    else
        l = '0', r = '9';

    for (char i = l; i <= r; i++)
    {
        temp.push_back(i);
        dfs(st + 1, i == l && start, i == r && ed, max(maxLast, i), min(minLast, i));
        temp.pop_back();
    }
}

void solve()
{
    int l, r;
    cin >> l >> r;
    a = to_string(l);
    b = to_string(r);
    res = a;
    ans = 10;
    temp.clear();

    if (a.size() + 1 <= b.size())
    {
        cout << string((int)a.size(), '9') << endl;
        return;
    }
    for (char i = a[0]; i <= b[0]; i++)
    {
        temp.push_back(i);
        dfs(1, i == a[0], i == b[0], i, i);
        temp.pop_back();
    }

    cout << res << endl;
}

```