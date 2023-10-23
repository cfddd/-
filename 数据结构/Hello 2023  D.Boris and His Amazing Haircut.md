## 题目
[Problem - D - Codeforces](https://www.cnblogs.com/cfddfc/p/17026079.html)
## 题意
- 给两个长度为n的数组a，b，再给一个数组长度为m的数组x，表示m次操作
- 每次操作把选择一个区间把a[l ~ r]中大于x[i]的变为x[j]，否则不变
- 问能否在m次操作内把数组a变为与数组b相等
## 思路
很容易知道，如果a[i] < b[i]，一定不可能
显然要把所有a数组，对于b数组相同元素进行操作，可以找到一个一个点 $ b[i] != a[i]$，关于b[i]
为了节省次数，我们要尽可能的合并这些点，把b[i]相同的点，转化为区间，区间数量要小于x数组中b[i]的数量
所以关键操作就在如何合并b[i]相同的点
可以用区间查询的策略，找出相邻两个位置，如果之间最大的值小于等于两个端点，那么这两个点可以合并，否则不能合并
实现可以通过st表，线段树，树状数组等（可恶啊，比赛的时候智商变低，一直在想着怎么能朴素优化，没想到这么简单）
可以通过单调栈，单调递减找出上一个大于等于当前的位置，如果栈顶为空或者大于当前点，这个点需要的数量就加一
如果等于，说明可以合并，跳过
 
## 区间查询O（nlogn+mlogm）
```cpp
const int N = 2e5+10,M = 20;
int a[N],b[N],x[N];

int st[N][M];

void ST(int n)//区间max
{
    //memset(st,0,sizeof st);
    for(int k = 0;k < M;k ++)
    {
        for(int i = 1;i <= n - (1<<k) + 1;i ++)
        {
            if(!k)st[i][k] = b[i];
            else{
                st[i][k] = max(st[i][k-1],st[i+(1<<(k-1))][k-1]);
            }
        }
    }
}
// int query(int l,int r)//区间max
// {
//     int t = M-1;
//     int res = 0;
//     while(t >= 0)
//     {
//         if(l + (1<<t) > r)t --;
//         else{
//             res = max(res,st[l][t]);
//             l += 1<<t;
//         }
//     }
//     return res;
// }
int query(int l, int r)
{
    int len = r - l + 1;
    int k = log(len) / log(2);
    return max(st[l][k], st[r - (1 << k) + 1][k]);
}
void solve() 
{
    map<int,vector<int>> tar;
    map<int,int> cnt;
    int n;cin >> n;
    fr(1,i,n+1)cin >> a[i];
    fr(1,i,n+1)cin >> b[i];
    int m;cin >> m;
    fr(1,i,m+1)cin >> x[i],cnt[x[i]] ++;
    
    fr(1,i,n+1)if(a[i] < b[i])
    {
        cout << "NO" << endl;
        return ;
    }
    fr(1,i,n+1)
    {
        if(a[i] != b[i])tar[b[i]].push_back(i);
    }
    
    ST(n);

    bool acc = 1;
    for(auto [key,y]:tar)
    {
        int num = 1;
        for(int i = 1;i < y.size();i ++)
        {
            //debug3(y[i-1],y[i],query(y[i-1],y[i]));
            if(query(y[i-1],y[i]) > key)num ++;
        }
        //debug4(key,num,cnt[key],y.size());
        if(num > cnt[key])acc = 0;
    }
    //for(auto [key,y]:tar)if(cnt[key])acc = 0;
    cout << (acc?"YES":"NO") << endl;
}

```
pps：st表第一次用，感觉有点像倍增

ps：`for(auto [key,y]:tar)if(cnt[key])acc = 0;`这样的用法居然在VScode里标错了，但是可以运行

## 单调栈（代码用了别人的，妙哉）
```cpp
#include<bits/stdc++.h>
using namespace std;
int T,n,a[200005],b[200005],m,x;

bool bb;
int main()
{
    scanf("%d",&T);
    while(T--){
        bb=0; map<int,int>mapp; stack<int>q;
        scanf("%d",&n);
        for(int i=1;i<=n;i++) scanf("%d",a+i);
        for(int i=1;i<=n;i++) scanf("%d",b+i);
        scanf("%d",&m);
        for(int i=1;i<=m;i++) scanf("%d",&x),mapp[x]++;
        for(int i=1;i<=n;i++){
            if(a[i]<b[i]){bb=1;break;}
            while(!q.empty()&&q.top()<b[i]) q.pop();
            if((q.empty()||(!q.empty()&&q.top()!=b[i]))&&a[i]>b[i]){
                q.push(b[i]);
                if(mapp[b[i]]<=0){bb=1;break;}
                mapp[b[i]]--;                    
            }
        }
        puts((bb?"NO":"YES"));
    }
}
 ```


 