## 题目
- https://ac.nowcoder.com/acm/contest/46812/H
## 题意
- 地图大小N*M，障碍物为“#”，地图上其他所有点有一个字母（“LRUD”之一，表示走的方向；“.”表示A停止）
- 有两个人A和B，A从（$x_t$，$y_t$）按照地图上的标记走，B从（$x_s$，$y_s$）走，A走一步，B走一步（B可以上下左右随意走或者停下）
- 给 q 个询问，每次询问是一对坐标（$x_t$，$y_t$），问从A这个点出发按照地图上标记的方向走，B最快多久遇上A
- 起点为（$x_s$，$y_s$）
## 思路
- 朴素的方法是$O（N * M * q）$
  1. 从（$x_s$，$y_s$）bfs，为每个位置标记上一个最短到达时间ti[i][j]
  2. 从（$x_t$，$y_t$）再次搜索，当遇到step >= ti[i][j]时，就是答案
- 朴素的方法显然不够过hard的，我们可以观察这个地图，发现从一个点走的空间变化和时间是正相关的（应该是这么说）
- 所以我们可以考虑用倍增记录
  1.$next[i * m+j][p]$表示从$（i，j）$出发，走$2^p$步的位置
  2.先根据地图，做出$next[i * m+j][0]$的位置
  3.然后就是经典的倍增转移啦
- 最后是经典的从最大的倍数逐倍缩小，直到不能缩小为止，这个最小的倍数就是答案
## 代码
```
#include <bits/stdc++.h>

using i64 = long long;

constexpr int dx[] = {0, 0, -1, 1};
constexpr int dy[] = {-1, 1, 0, 0};

int main() {
    std::ios::sync_with_stdio(false);
    std::cin.tie(nullptr);
    
    int n, m, xs, ys, q;
    std::cin >> n >> m >> xs >> ys >> q;
    
    std::vector<std::string> s(n);
    for (int i = 0; i < n; i++) {
        std::cin >> s[i];
    }
    
    std::vector dis(n * m, -1);
    
    std::queue<std::pair<int, int>> que;
    que.emplace(xs, ys);
    dis[xs * m + ys] = 0;
    
    while (!que.empty()) {
        auto [x, y] = que.front();
        que.pop();
        
        for (int k = 0; k < 4; k++) {
            int nx = x + dx[k];
            int ny = y + dy[k];
            if (s[nx][ny] != '#' && dis[nx * m + ny] == -1) {
                dis[nx * m + ny] = dis[x * m + y] + 1;
                que.emplace(nx, ny);
            }
        }
    }
    
    const int l = 20;
    
    std::vector nxt(l, std::vector(n * m, -1));
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            if (s[i][j] != '#') {
                int nx, ny;
                if (s[i][j] == '.') {
                    nx = i, ny = j;
                } else {
                    int k = std::string("LRUD").find(s[i][j]);
                    nx = i + dx[k];
                    ny = j + dy[k];
                    if (s[nx][ny] == '#') {
                        nx = i, ny = j;
                    }
                }
                nxt[0][i * m + j] = nx * m + ny;
            }
        }
    }
    
    for (int t = 1; t < l; t++) {
        for (int i = 0; i < n * m; i++) {
            if (nxt[t - 1][i] != -1) {
                nxt[t][i] = nxt[t - 1][nxt[t - 1][i]];
            }
        }
    }
    
    while (q--) {
        int x, y;
        std::cin >> x >> y;
        
        int u = x * m + y;
        if (dis[u] == -1) {
            std::cout << -1 << "\n";
        } else {
            int ans = 0;
            for (int t = 19; t >= 0; t--) {
                int v = nxt[t][u];
                if (dis[v] > ans + (1 << t)) {
                    ans += (1 << t);
                    u = v;
                }
            }
            ans++;
            std::cout << ans << "\n";
        }
    }
    
    return 0;
}
/*jiangly的代码*/
/*妙啊妙啊*/
```
