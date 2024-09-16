> [AC自动机wiki](https://oi-wiki.org/string/ac-automaton/#%E6%80%BB%E7%BB%93)
> 
> [力扣形成目标需要的的最小字符串数](https://leetcode.cn/problems/minimum-number-of-valid-strings-to-form-target-ii/)

## 思路
如果会AC自动机的话，很容易往这方面想

AC自动机的位置代表每次当前位置能匹配的前缀

如果产生回退，则说明对应字符串无法匹配，需要变更选择的前缀，即要改变次数

定义`f[i]`表示字符串在`i`以前变更所需要的次数

最简单的方法就是枚举`i`前面所有的点，再根据前缀是否存在来转移，但是这样显然时间复杂度不够

仔细观察可以发现：**距离i最近的点，是当前位置唯一的转移点**

> 为什么？因为再往前面其实就属于别的前缀里面了，没有转移的意义，所以我们找到距离i最近的能匹配上的就可以了

转移方程就是`f[i] = f[i - len[u]]`，其中`len[u]`表示当前匹配前缀字符串的长度
## 模版
需要kmp和trie基础
```cpp
class Solution {

    const static int N = 1e5 + 6;
    int n;

    int tr[N][26], tot;
    int e[N], fail[N];
    int depth[N],f[N];

    void insert(string &s) {
        int u = 0;
        for (int i = 0; i < s.size(); i++) {
            if (!tr[u][s[i] - 'a']) tr[u][s[i] - 'a'] = ++tot;  // 如果没有则插入新节点
            depth[tr[u][s[i] - 'a']] = depth[u] + 1;
            u = tr[u][s[i] - 'a'];                              // 搜索下一个节点

        }
        e[u]++;  // 尾为节点 u 的串的个数
    }

    void build() {
        queue<int> q;
        for (int i = 0; i < 26; i++)
            if (tr[0][i]) q.push(tr[0][i]);
        while (q.size()) {
            int u = q.front();
            q.pop();
            for (int i = 0; i < 26; i++) {
                if (tr[u][i]) {
                    fail[tr[u][i]] =
                        tr[fail[u]][i];  // fail数组：同一字符可以匹配的其他位置
                    q.push(tr[u][i]);
                } else {
                    tr[u][i] = tr[fail[u]][i];
                }

            }
        }
    }

    int query(char *t) {
        int u = 0, res = 0;
        for (int i = 1; t[i]; i++) {
            u = tr[u][t[i] - 'a'];  // 转移
            for (int j = u; j && e[j] != -1; j = fail[j]) {
                res += e[j], e[j] = -1;
            }
        }
        return res;
    }

public:
    int minValidStrings(vector<string>& words, string t) {
        for(auto &x:words){
            insert(x);
        }
        build();
        int u = 0, ans = 1;
        int n = t.size();
        for (int i = 0; i < n; i++) {
            int v = tr[u][t[i] - 'a'];  // 转移
            if(v == 0){
                return -1;  // 不存在
            }
            f[i + 1] = f[i - depth[v] + 1] + 1;
            u = v;
        }
        return f[n];
        
    }
};
```



