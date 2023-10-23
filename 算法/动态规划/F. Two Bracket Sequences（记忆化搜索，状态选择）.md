## 题目
<https://codeforces.com/contest/1272/problem/F>
## 题意
给两个括号字符串，找到一个括号序列合法的超字符串
## 思路

- [类似方法](https://leetcode.cn/problems/shortest-common-supersequence/solutions/2194615/cong-di-gui-dao-di-tui-jiao-ni-yi-bu-bu-auy8z/)

- 问题一：如何找到最短长度的值
  
- dp[i,j,k]表示s目前在匹配了第i个，前面匹配完了，j同理，k表示左括号比右括号多多少，超字符串的最小长度

- 从左往右考虑，如果放一个'('，如果字符串s或t可以匹配则向后走一位，对于')'同理

- 当s和t都匹配完了，需要判断
  - 如果k=0，那么这是一次合理的放置
  - 如果k!=0，那么这不是一次合理的方案，返回无穷大（无解）
- 使用记忆化搜索计算所有的dp状态

- 问题二：如何构造这个字符串
- 类似上面的记忆化状态，因为已经知道了所有状态的结果，可以想出一棵结果二叉树，即什么位置放什么，怎么走到叶子节点最近
- 第一个位置显然放'('，然后与s和t做匹配，
  - 如果k=0，那么这个位置只能放'('
  - 如果k!=0，那么这个位置既能放'('，又能放')'
- 走到终点结束，输出每次的选择

## [参考代码](https://codeforces.com/contest/1272/submission/208121767)

```cpp
const int N = 210;
 
int dp[N][N][2*N];
char s[N], t[N];
int n, m;
 
int dfs(int x, int y, int k) {
 if (k >= 2*N) return 0x3f3f3f3f;
 if (x == n && y == m && k == 0) return 0;
 if (~dp[x][y][k]) return dp[x][y][k];
 int &res = dp[x][y][k] = 0x3f3f3f3f;
 if (k) res = min(res, dfs(x + (s[x] == ')'), y + (t[y] == ')'), k - 1) + 1);
 res = min(res, dfs(x + (s[x] == '('), y + (t[y] == '('), k + 1) + 1);
 return res;
}
 
void rec(int x, int y, int k) {
 if (x == n && y == m && k == 0) return ;
 int res = dfs(x, y, k);
 if (k && dfs(x + (s[x] == ')'), y + (t[y] == ')'), k - 1) + 1 == res) {
  printf(")"), rec(x + (s[x] == ')'), y + (t[y] == ')'), k - 1);
  return ;
 }
 printf("("), rec(x + (s[x] == '('), y + (t[y] == '('), k + 1);
}

void solve()
{
 memset(dp, -1, sizeof(dp));
 scanf("%s%s", s, t), n = strlen(s), m = strlen(t);
 rec(0, 0, 0), printf("\n");
}
```
