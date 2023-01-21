# 数学知识
## 质数
### 试除法判定质数
```
void solve(int x){
	if(x < 2) return false;
    for (int i = 2; i <= x / i; i ++ ) // O(sqrt(n)); 2 * 3 = 6;
        if(x % i == 0) return false; 
    return true;
}
```
### 分解质因数
```
void solve(int x){
	for (int i = 2; i <= x / i; i ++ ){ // O(logn - sqrt(n)); 
        if(x % i == 0){ // i是x的质数
            int s = 0;
            while(x % i == 0) s ++, x /= i;
            printf("%d %d\n", i, s);
        }
    }
    if(x != 1) printf("%d %d\n", x, 1); //x中大于根号x的质因子
}
```
### 筛质数
**埃氏筛法**
```
void solve(int n){
	for (int i = 2; i <= n; i ++ )
        if(!st[i]){ 
	        primes[cnt ++] = i;
	        for (int j = i; j <= n; j += i) st[j] = true; //只需要把质数的倍数删掉
	    }
    printf("%d\n", cnt); // 质数的个数
}
```
**线性筛法**
```
void solve(int n){ //O(n),保证一个数只会被它的最小质因子筛掉
	for (int i = 2; i <= n; i ++ ){
        if(!st[i]) primes[cnt ++] = i;
        for (int j = 0; primes[j] <= n / i; j ++ ){
            st[primes[j] * i] = true;
            if(i % primes[j] == 0) break; 
        }
    }
    printf("%d\n", cnt);
}
```
#### 唯一分解定理，所有整数都可以由质数的次方获得

## 约数
**试除法**
```
void solve(){
	vector<int> p;
    for (int i = 1; i <= x / i; i ++ )
        if(x % i == 0){
            p.push_back(i);
            if(x / i != i) p.push_back(x / i); // i * i != x
        }
}
```
**约数个数**
```
void solve(){
	int n; cin >> n;
    unordered_map<int, int> mp; //用哈希表存储质因子的指数部分
    for (int i = 0; i < n; i ++ ){
        int x; cin >> x;
        for (int i = 2; i <= x / i; i ++ ) //算x的质因子
            while (x % i == 0) x /= i, mp[i] ++;
        
        if(x > 1) mp[x] ++; //存大于sqrt(x)的质因子
    }
    LL ans = 1;
    for (auto [k, v]: mp) ans = ans * (v + 1) % mod;
}
```
**约数之和**
```
void solve(){
	//...与上同
    LL ans = 1;
    for (auto [k, v]: mp){
        LL t = 1; //(p^0 + p^1 + ... + p^v)
        while (v -- ) t = (t * k + 1) % mod;
        ans = t * ans % mod;
    }
}
```
**最大公约数**
```
\\ d | a, d | b -> d | (ax + by);
int gcd(int a, int b)
{
    return b ? gcd(b, a % b) : a;
}
```
**最小公倍数**
```
a*b/gcd(a,b);
```
## 快速幂
```
LL qmi(LL a,int b,int p)//快速幂在于思想
{
    LL res=1;
    while(b)//对b进行二进制化,从低位到高位
    {
        if(b&1) res = res *a %p;
        b>>=1;
        a=a*a%p;
    }
    return res;
}
```
**快速幂求逆元**
```
//qmi(a, p - 2, p)
//费马小定理只能在m是质数是用
//b 存在乘法逆元的充要条件是 b 与模数 m 互质
//若m不一定是质数，要用欧几里得
LL qmi(int a, int b, int p)
{
    LL res = 1;
    while(b){
        if(b & 1) res = res * a % p;
        a = (LL)a * a % p;
        b >>= 1;
    }
    return res;
}
```

## 欧拉函数
![](./images/1662721970584.png)
```
	int n;cin >> n;
    for (int i = 0; i < n; i ++ ){
        int a;cin >> a;
        int ans = a;
        for(int j = 2;j <= a/j;j++){
            if(a % j == 0){
                while(a % j == 0) a = a/j;
                ans = ans / j * (j - 1);
            }
        }
        if(a > 1)ans = ans / a * (a - 1);
        cout << ans << endl;
    }
```

**筛法求欧拉函数**

```
int primes[N], cnt; //存储2~n中的所有质数
int p[N]; //存储每个数的欧拉函数p[1] = 1;
bool st[N];
long long solve(int x){
    long long ans = 0;
    p[1] = 1; 
    for (int i = 2; i <= x; i ++ ){  //线性筛法
        if(!st[i]){
            primes[cnt ++] = i;
            //如果是质数，那 1 ~ i-1都和质数i互质
            p[i] = i - 1; //所以与i互质的数有i - 1 个
        }
        for (int j = 0; primes[j] <= x / i; j ++ ){
            st[primes[j] * i] = true;
            
            if(i % primes[j] == 0){
                p[primes[j] * i] = primes[j] * p[i]; //与质数的种类有关,与质数的个数无关
                break; //优化，使成为线性的
            }
            //如果i%primes[j] != 0; primes[j]是i * primes[j]的最小质因子
            p[primes[j] * i] = (primes[j] - 1) * p[i];
        }
    }
    //求1-x之间的欧拉函数和
    for (int i = 1; i <= x; i ++ ) ans += p[i];
    return ans;
}
```
>欧拉定理:当a与n互质时, a^f(n) 同余于 1 (% n);
>费马定理:如果n是质数f(n) = n-1则: a^(n-1) 同余于 1 (% n);

## 组合数
**随时存**
```
//因为 1≤n≤10000, 1≤b≤a≤2000，所以可以把所有C(a, b)都处理出来
//C(a, b):从a个物品中选择b个 c[i][j]:从i个物品中选择j个
// (c[i - 1][j - 1] + c[i - 1][j]) % mod != c[i - 1][j - 1] % mod + c[i - 1][j] % mod
const int mod = 1e9 + 7, N = 2222;
int c[N][N];

void solve(){
    for (int i = 0; i <= 2000; i ++ )
        for (int j = 0; j <= i; j ++ ){
            if(!j) c[i][j] = 1;
            else c[i][j] = (c[i - 1][j - 1] + c[i - 1][j]) % mod; 
        }
}
```
**快速算简单组合数**
```
long long C(int n,int m){//n在100以内，m要选较小的一边
    long long res = 1;
    for(int i = 0; i < m; i++) res *= n - i, res /= i + 1;
    return res;
}
```
**处理阶乘逆元（用到过一次）**

```
vector<int> fac;
vector<int> ifac;

int binExp(int base, int exp) {
    base %= MOD;
    int res = 1;
    while (exp > 0) {
        if (exp & 1) {
            res = (int) ((long long) res * base % MOD);
        }
        base = (int) ((long long) base * base % MOD);
        exp >>= 1;
    }
    return res;
}

void precompute(int n) {
    fac.resize(n + 1);
    fac[0] = fac[1] = 1;
    for (int i = 2; i <= n; i++) {
        fac[i] = (int) ((long long) i * fac[i-1] % MOD);
    }

    ifac.resize(n + 1);
    for (int i = 0; i < fac.size(); i++) {
        ifac[i] = binExp(fac[i], MOD - 2);
    }
    return;
}

int nCr(int n, int r) {
    if ((n < 0) || (r < 0) || (r > n)) {
        return 0;
    }
    return (int) ((long long) fac[n] * ifac[r] % MOD * ifac[n - r] % MOD);
}
```

```
## II
//因为 1≤n≤10000, 1≤b≤a≤10^5 所以不能直接求,但是可以预处理出b！和a！和(a-b)！
//c[a][b] = (a!) / (b!)*((a-b)!) 预处理相应的阶乘和逆元阶乘
//因为除法取模与乘法取模不等价，所以需要换成逆元，求逆元的阶乘
//因为要mod 1e9+7(质数),所以用快速幂求
//每个式子前面都要加long long

typedef long long LL;
const int N = 1e5 + 10, mod = 1e9 + 7;
int fact[N], infact[N];
//快速幂求逆元
int qmi(int a, int b, int m){
    int ans = 1;
    while (b){
        if(b & 1)
            ans = (LL)a * ans % m;
        a = (LL)a * a % m;
        b >>= 1;
    }
    return ans;
}
void get_fact_infact(){
    fact[0] = 1; //跟总体计算有关，不影响计算的
    infact[0] = 1;
    for (int i = 1; i < N; i ++ ){
        fact[i] = (LL)fact[i - 1] * i % mod;
        infact[i] = (LL)infact[i - 1] * qmi(i, mod - 2, mod) % mod; // i的模mod的乘法逆元
    }
}
int main(){
    int n; cin >> n;
    get_fact_infact(); //阶乘和逆元阶乘
    while (n -- ){
        int a, b; cin >> a >> b;
        int ans = (LL)fact[a] * infact[b] % mod * infact[a - b] % mod;
        printf("%d\n", ans);
    }
    return 0;
}
```

```
## III
//1≤n≤20, 1≤b≤a≤10^18, 1≤p≤10^5 所以a,b用long long
//卢卡斯定理:c[a][b] = c[a%p][b%p] * c[a/p][b/p] (mod p) !!!!所以最终都会转化到10^5内
//可以不用fact[],infact[]数组，直接算就ok哒
// !!! 有*就要% ！！！！！

typedef long long LL;
int p;
int qmi(LL a, int b, int m){
    int ans = 1;
    while (b){
        if(b & 1) ans = (LL)ans * a % m;
        b >>= 1;
        a = (LL)a * a % m;
    }
    return ans;
}
int C(int a, int b, int p){
    if(b > a) return 0;
    int ans = 1;
    for (int i = 1, j = a; i <= b; i ++, j -- ){
        ans = (LL)ans * j % p;
        ans = (LL)ans * qmi(i, p - 2, p) % p;
    }
    return ans;
}
//卢卡斯定理
int lucas(LL a, LL b, int p){
    if(a < p && b < p) return C(a, b, p);
    return (LL)C(a % p, b % p, p) * lucas(a / p, b / p, p) % p; // !!!!!!!!!!!!!注意注意
}
int main(){
    int n; cin >> n;
    while (n -- ){
        LL a, b; cin >> a >> b >> p;
        int ans = lucas(a, b, p);
        printf("%d\n", ans);
    }
    return 0;
}
```
**## III**
```

//1≤n≤20, 1≤b≤a≤10^18, 1≤p≤10^5 所以a,b用long long
//卢卡斯定理:c[a][b] = c[a%p][b%p] * c[a/p][b/p] (mod p) !!!!所以最终都会转化到10^5内
//可以不用fact[],infact[]数组，直接算就ok哒
// !!! 有*就要% ！！！！！

typedef long long LL;
int p;
int qmi(LL a, int b, int m){
    int ans = 1;
    while (b){
        if(b & 1) ans = (LL)ans * a % m;
        b >>= 1;
        a = (LL)a * a % m;
    }
    return ans;
}
int C(int a, int b, int p){
    if(b > a) return 0;
    int ans = 1;
    for (int i = 1, j = a; i <= b; i ++, j -- ){
        ans = (LL)ans * j % p;
        ans = (LL)ans * qmi(i, p - 2, p) % p;
    }
    return ans;
}
//卢卡斯定理
int lucas(LL a, LL b, int p){
    if(a < p && b < p) return C(a, b, p);
    return (LL)C(a % p, b % p, p) * lucas(a / p, b / p, p) % p; // !!!!!!!!!!!!!注意注意
}
int main(){
    int n; cin >> n;
    while (n -- ){
        LL a, b; cin >> a >> b >> p;
        int ans = lucas(a, b, p);
        printf("%d\n", ans);
    }
    return 0;
}
```
**## IV**
```

//c[a][b] = a! / (b! * (a - b)!)
//求阶乘的质因子并将相应的次方相消，遍历每一项的质因子，依次相消，最后得出结果
//vector<int> 之间可以直接赋值
// a! = (a/p[1]) + (a/p[2]) + .... + (a/p[cnt]);

const int N = 5e3 + 10;
int prime[N], cnt;
bool st[N];
int num[N]; //记录对应的质因数最终出现的次数
void get_primes(int x){
    for (int i = 2; i <= x; i ++ ){
        if(!st[i]) prime[cnt ++] = i;
        for (int j = 0; prime[j] <= x / i; j ++ ){
            st[prime[j] * i] = true;
            if(i % prime[j] == 0) break;
        }
    }
}
int get(int a, int b){
    // a = 5, b = 2;  ans = 2, a = 2; ans = 3, a = 1; ans = 3, a = 0;
    int ans = 0;
    while (a) ans += a / b, a /= b;
    return ans;
}
//存储是个位数在前！
vector<int> mul(vector<int> ans, int x){
    vector<int> res;
    int t = 0; //表示余数
    for (int i = 0; i < (int)ans.size(); i ++ ){
        t += ans[i] * x;
        res.push_back(t % 10);
        t /=  10;
    }
    while (t) res.push_back(t % 10), t /= 10;
    return res;
}
int main(){
    int a, b; cin >> a >> b;
    get_primes(a);
    for (int i = 0; i < cnt; i ++ ){
        int p = prime[i];
        num[i] = get(a, p) - get(b, p) - get(a - b, p);
    }
    vector<int> ans;
    ans.push_back(1); // !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    for (int i = 0; i < cnt; i ++ ){
        //prime[i] 出现 num[i]次，所以计算num[i]次
        for (int j = 0; j < num[i]; j ++ ) ans = mul(ans, prime[i]);
    }
    int n = ans.size();
    for (int i = (int)ans.size() - 1; i >= 0; i -- ) printf("%d", ans[i]);
    return 0;
}
```

**## 满足条件的01序列**
```
>给定 n 个 0 和 n 个 1，它们将按照某种顺序排成长度为 2n 的序列，求它们能排列成的所有序列中，能够满足任意前缀序列中 0 的个数都不少于 1 的个数的序列有多少个

//换成二维格子图，0：向右移动；1：向上移动； 从起点(0, 0)到终点(n, n) 
//横坐标始终大于等于纵坐标才是合法路径
typedef long long LL;
const int mod = 1e9 + 7;

int qmi(int a, int b, int m){
    int ans = 1;
    while (b){
        if(b & 1) ans = (LL)ans * a % m;
        a = (LL)a * a % m;
        b >>= 1;
    }
    return ans;
}
int main(){
    int n; cin >> n;
    int a = n * 2, b = n;
    int ans = 1;
    for (int i = 1; i <= n; i ++ ){
        ans = (LL)ans * (a --) % mod;
        ans = (LL)ans * qmi(i, mod - 2, mod) % mod; 
    }
    ans = (LL)ans * qmi(n + 1, mod - 2, mod) % mod;
    printf("%d\n", ans);
    return 0;
}
```

## 博弈论
**## Nim游戏**
**结论：**
```
//必胜态：拿完之后可以剩下的状态变成先手必败态； 必败态：不管怎么操作都会变成先手必胜态
//定理：n堆(a[1], a[2], a[2] .... a[n])
// a[0]^a[1]^a[2]^a[3]...^a[n] = x /(x == 0) 先手必败态 / (x != 0)先手必胜态
// 先手操作完之后可以让^值变成0，即为先手必胜态
// 解：x != 0 : x的二进制表示中最高一位1在第k位，则必定存在a[i]的第k位是1，从a[i]中拿走a[i] - a[i]^x个石子(a[i]^x < a[i])
// 最后剩下：a[i] - (a[i] - a[i]^x) == a[i] ^ x;

const int N = 1e5 + 10;
int a[N];

int main(){
    int n; cin >> n;
    int x = 0;
    for (int i = 0; i < n; i ++ ){
        cin >> a[i];
        x ^= a[i];
    }
    if(x == 0) printf("No\n");
    else printf("Yes\n");
    return 0;
}
```
**## 台阶-Nim游戏**
```
//台阶Nim游戏：可以从任意台阶拿任意石头放到下一个台阶
//最后结束：当最后石头都到地上，台阶上都没有石头为止
//结束的上一步是：必胜的人从第1台阶把所有石头放到地上
//再上一步：必败的人把第2台阶上的石头都拿到第1台阶/
//结论：奇数台阶^=x;x==0为必败态，x!=0为必胜态
//因为可以通过移动奇数台阶上的石头实现x==0
//a[1]^a[3]^a[5]...^a[2*n-1] == x == 0;必败态
//证明：如果x!=0 一定可以移动一个奇数台阶的使x==0
//如果此==0 无论移动奇数台阶还是偶数台阶，最终都会迎来必胜者制造的x #F44336==
//如果x==0 无论移动奇数台阶还是偶数台阶，最终都会迎来必胜者制造的x==0必败局面
//必败者迎来的最后一个必败状态是第1^第3台阶==0

int main(){
    int n; cin >> n;
    int res = 0;
    for (int i = 1; i <= n; i ++ ){
        int x; cin >> x;
        if(i % 2) res ^= x;
    }
    
    if(res == 0) printf("No\n");
    else printf("Yes\n");
    
    return 0;
}
```
**## 集合-Nim游戏**
```
const int N = 111, M = 11111;
int f[M], a[N];
int k; 
int sg(int x){
    //如果f[x]被算过，就直接返回f[x],因为采用记忆化搜索，所以大大降低了时间复杂度
    //f数组是把最多10000个石子的状态存储下来
    if(f[x] != -1) return f[x];
    
    unordered_set<int> us;
    //用uset来存储所有的状态(起始的石子状态，和经过拿之后的石子状态)
    for (int i = 0; i < k; i ++ )
        if(x >= a[i]) us.insert(sg(x - a[i])); //递归
    
    //遍历当前图的当前层的所有已存状态，然后找到不存在的最小自然数
    for(int i = 0; ; i ++ ){
        if(!us.count(i)){
            f[x] = i;
            return f[x];
            //也可直接放在一起
            return f[x] = i;
        }
    }
}
int main(){
    cin >> k;
    for (int i = 0; i < k; i ++ ) cin >> a[i];
    memset(f, -1, sizeof f);
    int n; cin >> n;
    int res = 0;
    for (int i = 0; i < n; i ++ ){
        int x; cin >> x;
        res ^= sg(x);
    }
    if(!res) puts("No");
    else puts("Yes");
    
    return 0;
}
```
**## 拆分-Nim游戏**
```
//SG理论：多个独立局面的SG值等于这些局面SG值得异或值
//因为每次会分成规模更小得堆，所以一定会有结束。同样要记忆化搜索
//f数组存储每一个状态得sg结果

const int N = 111;
int f[N];
int sg(int x){
    if(f[x] != -1) return f[x];
    unordered_set<int> us;
    //i，j分别是另外两小堆得所有可能，j<=i是因为避免重复计算
    for (int i = 0; i < x; i ++ )
        for (int j = 0; j <= i; j ++ )
            us.insert(sg(i) ^ sg(j)); 
            //多个独立局面的sg值等于这些局面sg值得异或值
    
    //mex函数操作，找到最小的集合us中不存在得自然数
    for (int i = 0; ; i ++ )
        if(!us.count(i)){ 
	        return f[x] = i;
	    } 
}
int main(){
    int n; cin >> n;
    memset(f, -1, sizeof f);
    int res = 0;
    for (int i = 0; i < n; i ++ ){
        int x; cin >> x;
        res ^= sg(x);
    }
    if(res) puts("Yes");
    else puts("No");
    
    return 0;
}
```
# 数据结构
## 并查集
```
int find(int x)  // 并查集
{
    if (p[x] != x) p[x] = find(p[x]);
    return p[x];
}
```
### 带权并查集
有一个长度为 N 的整数序列。
下面会按顺序给出 M 个对该序列的描述，每个描述给定三个整数 l,r,s，表示该序列的第 l 个元素至第 r 个元素相加之和为 s。
对于每个描述，你需要判断该描述是否会与前面提到的描述发生冲突，如果发生冲突，则认为该描述是错误的。
如果一个描述是错误的，则在对后续描述进行判断时，应当将其忽略。
请你计算一共有多少个描述是错误的。
```
//带权并查集类似于前缀和
//如果给定区间的两个端点属于同一个并查集，判断这个区间的值是否与计算得到的值相等(必须是相等，不要考虑小于的情况)
//                    不属于同一个并查集，将这两个并查集合并。

#include <bits/stdc++.h>
using namespace std;
const int N = 2e5 + 10;
int p[N],w[N];

int find(int x)  // 并查集
{
    if (p[x] != x){
        int t = p[x];              //合并操作，带上w更新
        p[x] = find(p[x]);         //经过了状态压缩，头节点变成了别的节点的子
        w[x] += w[t];              //当前节点加上原先的头（现在是另一个的子），是当前节点到头的距离
    }
    return p[x];
}

int main()
{
    int n, m; cin >> n >> m;
    for (int i = 1; i <= n+1; i ++ ) {
        p[i] = i;
        w[i] = 0;
    }

    int ans = 0;
    while(m--)
    {
        int l,r,s;cin >> l >> r >> s;
        int ll = find(l);          //前缀和某一端加一都可
        int rr = find(r+1);
        if(ll == rr){
            if(w[l]-w[r+1] != s)ans++;//距离差值只能是等于s
        }else{
            p[ll] = rr;               //合并的方向，ll和rr可以交换（同时r+1和l交换）
            w[ll] = w[r+1]-w[l]+s;    //见图
        }
    }
    cout << ans << endl;
    //for(int i = 0;i <= n;i++)cout << w[i] << " ";

    return 0;
} 

```
动物王国中有三类动物 A,B,C，这三类动物的食物链构成了有趣的环形。
A 吃 B，B 吃 C，C 吃 A。
现有 N 个动物，以 1∼N 编号。
每个动物都是 A,B,C 中的一种，但是我们并不知道它到底是哪一种。
有人用两种说法对这 N 个动物所构成的食物链关系进行描述：
第一种说法是 1 X Y，表示 X 和 Y 是同类。
第二种说法是 2 X Y，表示 X 吃 Y。
此人对 N 个动物，用上述两种说法，一句接一句地说出 K 句话，这 K 句话有的是真的，有的是假的。
当一句话满足下列三条之一时，这句话就是假话，否则就是真话。
1. 当前的话与前面的某些真的话冲突，就是假话；
2. 当前的话中 X 或 Y 比 N 大，就是假话；
3. 当前的话表示 X 吃 X，就是假话。
你的任务是根据给定的 N 和 K 句话，输出假话的总数。
```
#include<iostream>
using namespace std;
const int N = 10e5 + 10;
int num[N],d[N];
int find(int x){
    if(num[x] != x){
        int t = find(num[x]);
        d[x] += d[num[x]];
        num[x] = t;
        
    }
    return num[x];
}
int main(){
    int n,m;cin >> n >> m;
    int z,x,y,ans = 0;
    for (int i = 1; i <= n; i ++ ) num[i] = i;
    for (int i = 0; i < m; i ++ ){
        scanf("%d%d%d", &z, &x,&y);
        if(x > n || y > n){ans++;continue;}
        int xx = find(x),yy = find(y);
        if(z==1){
            if(xx!=yy){
                num[xx]=yy;
                d[xx]=d[y]-d[x];
            }else if(xx == yy && (d[x]-d[y]) % 3 != 0)ans++;
        }else if(z == 2){
            if(xx!=yy){
                num[xx]=yy;
                d[xx]=d[y]-d[x]+1;
            }else if(xx == yy && (d[x]-d[y]-1)%3!=0)ans++;
        }
    }
    cout << ans;
}

```

## kmp
```
//kmp，找字串
char s[N],p[N];
int pre[N];                             //往前退
                                        //pre[j]的值每次最多加1
                                        //模式串的最后一位字符不影响pre数组的结果
int main(){
    int n,m;
    cin >> n >> (p + 1) >> m >> (s + 1);
                                        //把前缀和后缀相等的下标存入ne
    for(int i = 2,j = 0;i <= n;i++){
        while(j && p[i] != p[j + 1])j = pre[j];//往前匹配为止
        if(p[i] == p[j + 1]) j++;
        pre[i] = j; 
    }
    //kmp
    for(int i = 1,j = 0;i <= m;i++){
        while(j && p[j + 1] != s[i]) j = pre[j];//往前匹配为止
        if(p[j + 1] == s[i]) j++;       //往后
        if(j == n){                     //匹配到了
            cout << i - n << " ";
            j = pre[j];
        }
    }
}
```
## trie字典树
**在集合中查找一个字符串出现的次数**
```
trie字符串树

const int N = 100010;

int son[N][26], cnt[N], idx;
char str[N];

void insert(char *str)//用数组模拟这个26叉树
                    //功能为往集合中插入一个字符串
{
    int p = 0;
    for (int i = 0; str[i]; i ++ )
    {
        int u = str[i] - 'a';
        if (!son[p][u]) son[p][u] = ++ idx;
        p = son[p][u];
    }
    cnt[p] ++ ;//p没有重复，见idx
}

int query(char *str)//在集合中查找一个字符串出现的次数
{
    int p = 0;
    for (int i = 0; str[i]; i ++ )
    {
        int u = str[i] - 'a';
        if (!son[p][u]) return 0;
        p = son[p][u];
    }
    return cnt[p];
}
```
**字典树的按位运用**
```
//找出数组中任意两个异或结果最大的
const int N = 100010, M = 3100010;//M接近N的30倍，n最大是N，N个数字每个存30位idx

int n;
int a[N], son[M][2], idx;

void insert(int x)                //按照二进制安排树
{
    int p = 0;
    for (int i = 30; i >= 0; i -- )
    {
        int &s = son[p][x >> i & 1];//元素从首位开始各位
        if (!s) s = ++ idx;        //s“引用”son[p][x >> i & 1] ,s改变了，节点也跟着改变
        p = s;
    }
}

int search(int x)
{
    int p = 0, res = 0;
    for (int i = 30; i >= 0; i -- )//从最高位开始，找son[p][!s]是否存在（这样让最高位的异或结果是1最大）
    {                              //如此往复直到最后一位
        int s = x >> i & 1;
        if (son[p][!s])
        {
            res += 1 << i;         //成功找到，res加上对应的数
            p = son[p][!s];        //下一层
        }
        else p = son[p][s];        //走原来的下一层
    }
    return res;
}
```
##  线段树or 树状数组
**树状数组动态维护前缀和**
```
#include <iostream>
#include <cstring>
#include <algorithm>
using namespace std;
const int N = 1e5+10;
int a[N];
int n,m;

int lowbit(int x)  // 返回末尾的1
{
    return x & -x;
}

void add(int x,int w){
    for(int i = x;i <= n;i += lowbit(i)){
        a[i] += w;
    }
}

int query(int u){
    int res = 0;
    for(int i = u;i > 0;i -= lowbit(i)){
        res += a[i];
    }
    return res;
}

int main()
{
    cin >> n >> m;
    for (int i = 1; i <= n; i ++ ){
        int t;cin >> t;
        add(i,t);
    }
    for(int i = 1;i <= m;i ++){
        int k,a,b;cin >> k >> a >> b;
        if(k == 1){
            add(a,b);
        }else{
            cout << query(b) - query(a-1) << endl;
        }
    }
    return 0;
}
```
**线段树动态维护前缀和**
```
#include <iostream>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 1e5+10;
int a[N];
int n,m;
struct node{
    int l,r,sum;
}nodes[N*4];

void pushup(int u){
    nodes[u].sum = nodes[u << 1].sum + nodes[u << 1 | 1].sum;
}

void build(int u,int l,int r){
    if(l == r)nodes[u] = {l,r,a[r]};
    else{
        nodes[u] = {l,r};
        int mid = l + r >> 1;
        build(u << 1,l,mid);
        build(u << 1 | 1,mid + 1,r);
        pushup(u);
    }
}

int query(int u,int l,int r){
    if(l <= nodes[u].l && nodes[u].r <= r)return nodes[u].sum;
    int mid = nodes[u].l + nodes[u].r >> 1;
    int sum = 0;
    if(mid >= l)sum += query(u << 1,l,r);
    if(mid < r)sum += query(u << 1 | 1,l,r);
    return sum;
}

void add(int u,int x,int w){
    if(nodes[u].l == nodes[u].r)nodes[u].sum += w;
    else{
        int mid = nodes[u].l + nodes[u].r >> 1;
        if(mid >= x)add(u << 1,x,w);
        else add(u << 1 | 1,x,w);
        pushup(u);
    }
}

int main()
{
    cin >> n >> m;
    for (int i = 1; i <= n; i ++ )cin >> a[i];
    
    build(1,1,n);
    
    int k,a,b;
    for(int i = 0;i < m; i++){
        cin >> k >> a >> b;
        if(k){
            add(1,a,b);
        }else{
            printf("%d\n",query(1,a,b));
        }
    }
}
```
**线段树最大子段板子**
```

#define lson i << 1
#define rson i << 1 | 1
 
struct node{
	ll l, r;
	ll msum, lsum, rsum, sum;
    bool flag;
}tree[maxn<<2];
 
int n;
 
void push_up(int i){
    tree[i].msum = max(max(tree[lson].msum, tree[rson].msum), tree[lson].rsum + tree[rson].lsum);
    tree[i].lsum = max(tree[lson].lsum,  tree[lson].sum + tree[rson].lsum);
    tree[i].rsum = max(tree[rson].rsum, tree[rson].sum + tree[lson].rsum);
    if(tree[lson].sum > 0 && tree[rson].sum > 0)tree[i].sum = tree[lson].sum + tree[rson].sum;
    else tree[i].sum = -1e18;
}
 
void build(int i, int l, int r){
	tree[i].l = l, tree[i].r = r;
	if(l == r) { tree[i].msum = tree[i].lsum = tree[i].rsum = tree[i].sum = 0; return ; }
	int mid = (l + r) >> 1;
	build(lson, l, mid);
	build(rson, mid + 1, r);
	push_up(i);
}
 
void update(int i, int pos, int val){
    
	if(tree[i].l == tree[i].r) { 
        if(val != -1) {
            tree[i].msum += val;  tree[i].lsum += val; tree[i].rsum += val; tree[i].sum += val; return; 
        }
        else {
            tree[i].msum = -1e18;
            tree[i].lsum = -1e18;
            tree[i].rsum = -1e18;
            tree[i].sum = -1e18;
            return;
        }
    }
	int mid = (tree[i].l + tree[i].r) >> 1;
	if(pos <= mid) update(lson, pos, val);
	else update(rson, pos, val);
	push_up(i);
}
 
node Query(int i, int l, int r){
	if(tree[i].l >= l && tree[i].r <= r) return tree[i];
	int mid = (tree[i].l + tree[i].r) >> 1;
	if(r <= mid) return Query(lson, l, r);
	else if(l > mid) return Query(rson, l, r);
	else {
		node left = Query(lson, l, mid), right = Query(rson, mid + 1, r);
		node ret;
		ret.sum = left.sum + right.sum;
		ret.msum = max(max(left.msum, right.msum), left.rsum + right.lsum);
		ret.lsum = max(left.lsum, left.sum + right.lsum);
		ret.rsum = max(right.rsum, right.sum + left.rsum);
		return ret;
	}
}
```
## 字符串hash
**单hash + 自然溢出**
```
/*
//字符串hash，普通的自然溢出（容易被刻意卡）
typedef unsigned long long ULL;
const int N = 1e5+5,P = 131;//131 13331
ULL h[N],p[N];
// h[i]前i个字符的hash值
// 字符串变成一个p进制数字，体现了字符+顺序，需要确保不同的字符串对应不同的数字
// P = 131 或  13331 Q=2^64，在99%的情况下不会出现冲突
ULL query(int l,int r){
    return h[r] - h[l-1]*p[r-l+1];
}
int main()
{
    int n,m;scanf("%d %d",&n,&m);
    char *str = new char[n]; scanf("%s",str + 1);
    p[0] = 1;
    for (int i = 1; i <= n; i ++ ){
        h[i] = h[i - 1] * P + str[i];
        p[i] = p[i - 1] * P;
    }
    while (m -- ){
        int l1,r1,l2,r2;
        cin >> l1 >> r1 >> l2 >> r2;
        if(query(l1,r1) == query(l2,r2)) cout << "Yes" << endl;
        else cout << "No" << endl;
    }
}
*/
```

**双hash + 马拉车**
```
#include <bits/stdc++.h>

#define all(x) (x).begin() , x.end() 
// #define f first 
// #define s second 
using namespace std; 
typedef long long ll; 
typedef unsigned long long ull ; 
typedef pair<ll, ll> pll; 
typedef pair<long, int> pli; 
const int N = 1e6 + 10 , M = 150 , mod2 = 1222827239 , P = 1331 , Q = 197; 
ll mod = 1E12 + 39 ; 

int n , m, L[N];
char s[N], str[N] ; 
ll h[N] , p[N] , hh[N] , pp[N] ;
map<pll,int> mp ; 

ll get(int l , int r){
    return (h[r] - __int128(h[l - 1]) * p[r - l + 1] % mod + mod) % mod;
}
ll get2(int l , int r){
    return (hh[r] - hh[l - 1] * pp[r - l + 1] % mod2 + mod2) % mod2;
}

void getstr() {
	int k = 0;
	str[k++] = '@';
	for (int i = 0; i < n; i++) 
	    str[k++] = '#' , str[k++] = s[i];
	
	str[k++] = '#' ;  
	n = k;
	str[k] = 0 ; 
}

void manacher(int u) {
	int mx = 0, id; 
	for (int i = 1; i < n; i++) {
		if (i < mx) L[i] = min(mx - i, L[2 * id - i]);
		else L[i] = 1; 
		
		while (str[i + L[i]] == str[i - L[i]]) 
		{
		    ll t1 = get(i - L[i], i + L[i]) , t2 = get2(i - L[i], i + L[i]); 
		    t2 = 0 ; 
		    if(str[i + L[i]] == '#' && mp[{t1,t2}] == u - 1) {
		        mp[{t1,t2}] ++ ; 
		    }
		    L[i]++;
		}
		
		if (L[i] + i > mx) {
			mx = L[i] + i;
			id = i;
		}
	}
}

int solve() 
{
    cin >> m ;  
    for(int i = 1 ; i <= m ; i ++)
    {
        cin >> s ; n = strlen(s) ; 
        
        getstr() ; 
        p[0] = pp[0] = 1 ; h[0] = hh[0] = 0 ; 
        for( int i = 1 ; i <= n ; i ++)
        {
            h[i] = (h[i - 1] * P + str[i]) % mod;
            p[i] = p[i - 1] * P % mod;   
            hh[i] = (hh[i - 1] * P + str[i]) % mod2 ;
            pp[i] = pp[i - 1] * P % mod2 ; 
        }
        manacher(i); 
    }
    
    ll ans = 0 ;
    for(auto [x,c] : mp) if(c == m) ans ++ ; 
    cout << ans << endl ; 
    
    return 1 ; 
}
int main()
{
    ios::sync_with_stdio(0);cin.tie(0);cout.tie(0);
	int tt = 1; //cin >> tt;
    
	for(int i = 1 ; i <= tt ; i ++ )
	{
	    solve() ;
	    //cout << solve() << endl ; 
		//cout << (solve()? "YES\n" : "NO\n" ) ; 
	}
	return 0;
}

```
## 中心拓展算法
为马拉车算法做准备
但是马拉车算法找的是最长的回文子串
要找出所有的回文字串，还是要用这个中心拓展
```
//中心拓展算法
void countSubstrings(string s,int kk) {
 
        int n = s.size(), ans = 0;
        for (int k = 0; k < 2 * n - 1; ++k)
        {
            int i = k / 2, j = k / 2 + k % 2;
 
            //满足边界条件，且s[i] == s[j]，则代表一个新的回文字符串的诞生，否则，跳出循环
            while ( i >= 0 && j < n && s[i] == s[j] )
            {
                if(j - i + 1 >= kk)pa[ed++] = {i,j};
                --i;
                ++j;
            }
        }
    }
```
## 马拉车
```
马拉车算法O（N）找最长回文串
string Mannacher(string s)
{
    //插入"#"
    string t="$#";
    for(int i=0;i<s.size();++i)
    {
        t+=s[i];
        t+="#";
    }
    
    vector<int> p(t.size(),0);
    //mx表示某个回文串延伸在最右端半径的下标，id表示这个回文子串最中间位置下标
    //resLen表示对应在s中的最大子回文串的半径，resCenter表示最大子回文串的中间位置
    int mx=0,id=0,resLen=0,resCenter=0;

     //建立p数组
    for(int i=1;i<t.size();++i)
    {
        p[i]=mx>i?min(p[2*id-i],mx-i):1;

        //遇到三种特殊的情况，需要利用中心扩展法
        while(t[i+p[i]]==t[i-p[i]])++p[i];

        //半径下标i+p[i]超过边界mx，需要更新
        if(mx<i+p[i]){
            mx=i+p[i];
            id=i;
        }

        //更新最大回文子串的信息，半径及中间位置
        if(resLen<p[i]){
            resLen=p[i];
            resCenter=i;
        }
        //全部回文串在原位置对应
        //if(p[i]-1)cout << s.substr((i-p[i])/2,p[i]-1) << endl;
    }

    //最长回文子串长度为半径-1，起始位置为中间位置减去半径再除以2
    return s.substr((resCenter-resLen)/2,resLen-1);
}
```
# 图论
## 建图
```
//#pragma GCC optimize(3,"Ofast","inline")
//#pragma GCC optimize(2)

//int h[N], e[M], ne[M], idx;
//int h[N], e[M], w[M], ne[M], idx;

// void add(int a,int b){
// 	e[idx] = b;ne[idx] = h[a];h[a] = idx++;
// }
//void add(int a,int b,int w){
//	e[idx] = b;w[idx] = w;ne[idx] = h[a];h[a] = idx++;
//}
```
### 拓扑顺序
```
const int N = 100010;

int n, m;
int h[N], e[N], ne[N], idx;
int d[N];
int q[N];

void add(int a, int b)//邻接表
{
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++ ;
}

bool topsort()
{
    int hh = 0, tt = -1;//模拟队列

    for (int i = 1; i <= n; i ++ )//入队  入度为0的下标
        if (!d[i])
            q[ ++ tt] = i;

    while (hh <= tt)
    {
        int t = q[hh ++ ];//出队

        for (int i = h[t]; i != -1; i = ne[i])//按链上走
        {
            int j = e[i];//判断下一节点在入度减一后是否要入队
            if (-- d[j] == 0)
                q[ ++ tt] = j;
        }
    }

    return tt == n - 1;//有向无环图全部入队，tt == n - 1
}

int main()
{
    scanf("%d%d", &n, &m);

    memset(h, -1, sizeof h);

    for (int i = 0; i < m; i ++ )
    {
        int a, b;
        scanf("%d%d", &a, &b);
        add(a, b);

        d[b] ++ ;
    }

    if (!topsort()) puts("-1");
    else
    {
        for (int i = 0; i < n; i ++ ) printf("%d ", q[i]);
        puts("");
    }

    return 0;
}
```
## dijkstra邻接表版
```
const int N = 510;

int n, m;
int g[N][N];
int dist[N];
bool st[N];

int dijkstra()  // 求1号点到n号点的最短路距离，如果从1号点无法走到n号点则返回-1
{
    memset(dist, 0x3f, sizeof dist);
    dist[1] = 0;
    for (int i = 0; i < n - 1; i ++ )//n次
    {
        int t = -1;
        for (int j = 1; j <= n; j ++ )
            if (!st[j] && (t == -1 || dist[t] > dist[j]))
                t = j;//当前的最短边

        for (int j = 1; j <= n; j ++ )
            dist[j] = min(dist[j], dist[t] + g[t][j]);//更新最短距离

        st[t] = true;
    }

    if (dist[n] == 0x3f3f3f3f) return -1;
    return dist[n];
}


int main()
{
    scanf("%d%d", &n, &m);

    memset(g, 0x3f, sizeof g);
    while (m -- )
    {
        int a, b, c;
        scanf("%d%d%d", &a, &b, &c);

        g[a][b] = min(g[a][b], c);//邻接矩阵a到b
    }

    printf("%d\n", dijkstra());
    //cout << 1061109567  - 0x3f3f3f3f;

    return 0;
}
```
## dijkstra堆优化版
```
const int N = 150010;
typedef pair<int, int> PII;
int n, m;
int e[N],w[N],ne[N],h[N],idx = 0;
int dist[N];
bool st[N];
void add(int a, int b, int c)  // 添加一条边a->b，边权为c
{
    e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx ++ ;
}

int dijkstra()  // 求1号点到n号点的最短路距离，如果从1号点无法走到n号点则返回-1
{
    memset(dist, 0x3f, sizeof dist);
    dist[1] = 0;
    priority_queue<PII,vector<PII>,greater<PII> > heap;//小根堆pair<权，点>,按权排序
    heap.push({0,1});
    while(!heap.empty()){
        auto t = heap.top();
        heap.pop();
        
        int ww = t.first,ee = t.second;    //pair<权，点>
        if(st[ee]) continue;
        st[ee] = true;
        
        for(int i = h[ee];i != -1;i = ne[i]){
            int j = e[i];
            if (dist[j] > dist[ee] + w[i])
            {
                dist[j] = dist[ee] + w[i];
                heap.push({dist[j], j});
            }
        }
    }
    if (dist[n] == 0x3f3f3f3f) return -1;
    return dist[n];
}


int main()
{
    scanf("%d%d", &n, &m);

    memset(h, -1, sizeof h);
    while (m -- )
    {
        int a, b, c;
        scanf("%d%d%d", &a, &b, &c);

        add(a, b, c);
    }

    printf("%d\n", dijkstra());
    //cout << 1061109567  - 0x3f3f3f3f;

    return 0;
}
```
## bellman-ford——有负权边的最短路
```
const int N = 10010;
struct node{
    int a,b,c;
}nodes[N];
int n,m,k;
int dist[510],cpy[520];

void bellman_ford(){
    
    memset(dist,0x3f,sizeof dist);
    dist[1] = 0;
    for(int i = 0;i < k;i++){                             //只走不超过k条边
        memcpy(cpy,dist,sizeof dist);                     //防止串联
        for(int j = 0;j < m;j++){                         //按上一步的结果来更新下一步的最短步数
            struct node xx = nodes[j];                    
            dist[xx.b] = min(dist[xx.b],cpy[xx.a] + xx.c);
        }
    }
}

int main()
{
    cin >> n >> m >> k;
    for (int i = 0; i < m; i ++ )
    cin >> nodes[i].a >> nodes[i].b >> nodes[i].c;
    
    bellman_ford();
    
    if(dist[n] > 0x3f3f3f3f/2)cout << "impossible";          //500 * 10000全是负的也不会超过这个1e9/2
    else cout << dist[n];
}
```
## spfa求最短路，边权可以为负数
```
const int N = 150010;
typedef pair<int, int> PII;
int n, m;
int e[N],w[N],ne[N],h[N],idx = 0;
int dist[N];
bool st[N];
void add(int a, int b, int c)  // 添加一条边a->b，边权为c
{
    e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx ++ ;
}

int spfa()  // 求1号点到n号点的最短路距离，如果从1号点无法走到n号点则返回-1
{
    memset(dist, 0x3f, sizeof dist);
    dist[1] = 0;
    queue<int> dd;
    dd.push(1);
    while(!dd.empty()){
        int k = dd.front();
        dd.pop();
        st[k] = false;
        for(int i = h[k];i != -1;i = ne[i]){
            int point = e[i];
            if(dist[point] > dist[k] + w[i]){
                dist[point] = dist[k] + w[i];
                if(!st[point]){
                    dd.push(point);
                    st[point] = true;
                }
            }
        }
    }
    return dist[n];
}


int main()
{
    scanf("%d%d", &n, &m);

    memset(h, -1, sizeof h);
    while (m -- )
    {
        int a, b, c;
        scanf("%d%d%d", &a, &b, &c);

        add(a, b, c);
    }

    int ans = spfa();
    if(ans == 0x3f3f3f3f) cout << "impossible";
    else cout << ans;
    //cout << 1061109567  - 0x3f3f3f3f;

    return 0;
}
```
## spfa判断是否存在负权环
**即图中存在一个环，一直绕着他走，加权只会越来越小**
```
const int N = 150010;
typedef pair<int, int> PII;
int n, m;
int e[N],w[N],ne[N],h[N],idx = 0;
int dist[N];
bool st[N];
void add(int a, int b, int c)  // 添加一条边a->b，边权为c
{
    e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx ++ ;
}

int spfa()  // 求1号点到n号点的最短路距离，如果从1号点无法走到n号点则返回-1
{
    memset(dist, 0x3f, sizeof dist);
    dist[1] = 0;
    queue<int> dd;
    dd.push(1);
    while(!dd.empty()){
        int k = dd.front();
        dd.pop();
        st[k] = false;
        for(int i = h[k];i != -1;i = ne[i]){
            int point = e[i];
            if(dist[point] > dist[k] + w[i]){
                dist[point] = dist[k] + w[i];
                if(!st[point]){
                    dd.push(point);
                    st[point] = true;
                }
            }
        }
    }
    return dist[n];
}


int main()
{
    scanf("%d%d", &n, &m);

    memset(h, -1, sizeof h);
    while (m -- )
    {
        int a, b, c;
        scanf("%d%d%d", &a, &b, &c);

        add(a, b, c);
    }

    int ans = spfa();
    if(ans == 0x3f3f3f3f) cout << "impossible";
    else cout << ans;
    //cout << 1061109567  - 0x3f3f3f3f;

    return 0;
}
```
## Floyd求多源最短路，边权可以为负数
**图中不存在负权回路**
```
const int N = 210;
int d[N][N];
int n,m,k,INF = 0x3f3f3f3f;

void floyd(){
    for(int k = 1;k <= n;k++){
        for (int i = 1; i <= n; i ++ ){
            for (int j = 1; j <= n; j ++ ){
                d[i][j] = min(d[i][j],d[i][k] + d[k][j]);//dp
            }
        }
    }
}
int main()
{
    cin >> n >> m >> k;
    for (int i = 1; i <= n; i ++ ){
        for(int j = 1; j <= n;j ++){
            if(i == j)d[i][j] = 0;
            else d[i][j] = INF;
        }
    }
    int x,y,z;
    for(int i = 1;i <= m;i++){
        
        cin >> x >> y >> z;
        d[x][y] = min(d[x][y],z);                          //保留最小的
    }
    floyd();
    for (int i = 1; i <= k; i ++ ){
        cin >> x >> y;
        if(d[x][y] >= INF-1e6)cout << "impossible" << endl;
        else cout << d[x][y] << endl;
    }
    
}
```
## Prim算法求最小生成树
**n  个顶点和 n−1 条边构成的无向连通子图被称为 G 的一棵生成树
其中边的权值之和最小的生成树被称为无向图 G 的最小生成树**
```
int n,m;
int g[N][N];
int dist[N];
bool st[N];
//Dijkstra算法是更新到起始点的距离，Prim是更新到集合dist的距离
int prim()
{
    int res = 0;
    memset(dist,0x3f,sizeof(dist));
    dist[1] = 0;
    for(int i= 0;i < n;i ++)
    {
        int t = -1;
        for(int j = 1;j <= n;j++)
        {
            if(!st[j] && (t == -1 || dist[t] > dist[j]))t = j;
        }
        
        if(i && dist[t] == 0x3f3f3f3f)return 0x3f3f3f3f;
        
        if(i)res += dist[t];
        
        for(int j = 1;j <= n;j ++)
        {
            dist[j] = min(dist[j],g[t][j]);
        }
        st[t] = true;
    }
    return res;
}

int main()
{
    cin >> n >> m;
    memset(g,0x3f,sizeof(g));
    for(int i = 0;i < m;i++)
    {
        int u,v,w;
        cin >> u >> v >> w;
        g[u][v] = g[v][u] = min(g[u][v],w);
    }
    
    int ans = prim();
    ans == 0x3f3f3f3f?cout << "impossible" : cout << ans;
}
```
## kruskal算法求最小生成树
```
pair<int,pair<int,int> > edge[200010];//{w,{u,v}}
int p[100010];

int find(int x)  // 并查集
{
    if (p[x] != x) p[x] = find(p[x]);
    return p[x];
}


int main()
{
    int n,m;cin >> n >> m;
    for (int i = 0; i < m; i ++ )
    cin >> edge[i].second.first >> edge[i].second.second >> edge[i].first;
    
    sort(edge,edge + m);
    
    for(int i = 1;i <= n;i++) p[i] = i;
    
    int ans = 0,cnt = 0;                
                                        //kruskal算法
                                        //因为线段权重已经排好了序，所以一定是边权较小的独立的点先联通
                                        //到达了最小生成树的目的
                                        //在此之后的重复访问点都在同一个并查集内，被无视
    for(int i = 0;i < m;i++)
    {
        int a = find(edge[i].second.first),b = find(edge[i].second.second);
        if(a != b)
        {
            p[a] = b;
            ans += edge[i].first;
            cnt++;
        }
    }
    
    if(cnt < n - 1)cout << "impossible";//所有点“联通”后，联通数是总点数n的n-1，以此来判断是否有连通图
    else cout << ans;
}
```
## 染色法判断二分图
**二分图：给定无向无权图，分成两部分，相邻的点不在同一个部分当中
该图不可能含有点总数为奇数的环
**

```
//st中的1  2表示两种颜色，初始时0没有颜色

const int N = 100010,M = 200010;
int h[N], e[M], ne[M], idx;
int st[N];

void add(int a, int b)  // 添加一条边a->b
{
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++ ;
}

bool dfs(int u,int color){
    st[u] = color;
    for(int i = h[u];i != -1;i = ne[i]){
        int j = e[i];
        if(!st[j] && !dfs(j,3 - color))return false;//相邻点之间的颜色肯定是不一样的
        else if(st[j] == color)return false;        //如果搜索到相邻点与当前点的颜色相同，退出
    }
    return true;
}

int main()
{
    int n,m;cin >> n >> m;
    
    memset(h, -1, sizeof h);//这句...
    
    for(int i = 0;i < m;i++){
        int u,v;
        scanf("%d%d", &u, &v);
        add(u,v);
        add(v,u);
    }
    bool flag = true;
    for (int i = 1; i <= n; i ++ ){
        if(!st[i] && !dfs(i,1)){                      //从任何一个节点开始涂颜色1，确保所有结点
            flag = false;                             //对他的邻点已涂色或者刚涂色上都搜索过，最终输出是否是二分图
            break;
        }
    }
    if(flag)puts("Yes");
    else puts("No");
}
```
## 匈牙利算法
```
匈牙利算法，二分图的最大匹配
// 最坏O（n^2）
const int N = 100010;
int h[510], e[N], ne[N], idx;
bool st[N];     //每次走的状态
int match[N];   //女生的男友

void add(int a,int b){
    e[idx] = b;ne[idx] = h[a];h[a] = idx++;
}

bool find(int man){
    for(int i = h[man];i != -1;i = ne[i]){
        int j = e[i];
        if(!st[j]){
            st[j] = true;
            if(!match[j] || find(match[j])){//女生没有男友 或者 找这女生匹配男生的是否还有选择
                match[j] = man;
                return true;
            }
            
        }
    }
    return false;
}

int main()
{
    memset(h, -1, sizeof h);
    int n1,n2,m;cin >> n1 >> n2 >> m;
    for(int i = 0;i < m;i++){
        int a,b;
        cin >> a >> b;
        add(a, b);
    }
    
    int ans = 0;
    for(int i = 1;i <= n1;i++){
        memset(st, 0, sizeof st);
                                    // 
                                    // 每次归零：如果不归零，前人看上的妹子，后人再选就会发现不能选
                                    // 所以重置后，前人看上的妹子后人再选，进入递归，假设这一次st标记前一个妹子被第二个人选择
                                    // 前人看不了原配，找其他的选择，如果找到了，返回成功
                                    // 
        if(find(i))ans++;
    }
    cout << ans;
}
```
## LCA-倍增法
```
const int N = 4e4+10,M = N*2;
int h[N], e[M], ne[M], idx;
int depth[N],f[N][16];
int root;

void add(int a, int b)  // 添加一条边a->b
{
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++ ;
}

void bfs(int u)
{
    memset(depth,0x3f,sizeof depth);
    queue<int> q;
    q.push(u);
    depth[u] = 1;depth[0] = 0;                          //设置0的深度的原因为，往上到祖先的过程很容易出现超出根节点的情况、
                                                        //比如lca（root,root）,显然会出现错误
    while(q.size())                                     //按照树的结构bfs
    {
        int t = q.front();q.pop();
        
        for(int i = h[t];i != -1;i = ne[i])
        {
            int j = e[i];                               //当前节点的子节点
            if(depth[j] > depth[t] + 1)                 //只往深度更深的搜
            {
                depth[j] = depth[t] + 1;
                f[j][0] = t;                            //子节点往上的2^0的祖先
                q.push(j);
                for(int k = 1;k <= 15;k ++)             //倍增思想（类似递推）：节点的2^k的祖先 = 节点的2^(k-1)的祖先的2^(k-1)的祖先
                {
                    f[j][k] = f[f[j][k-1]][k-1];
                }
            }
        }
    }
}

int lca(int a,int b)
{
    if(depth[a] < depth[b])swap(a,b);                   //方便对齐
    for(int k = 15;k >= 0;k --)                         //让a的深度和b相同
    {
        if(depth[f[a][k]] >= depth[b])
        {
            a = f[a][k];
        }
    }
    
    if(a == b)
    {
        return a;
    }
    
    for(int k = 15;k >= 0;k --)                         //往上倍减的第一个共同祖先就是lca
    {
        if(f[a][k] != f[b][k])
        {
            a = f[a][k];
            b = f[b][k];
        }
    }
    return f[a][0];
}


//bfs(root);这样开始从图在f数组中用bfs建立关系
```
## 树上差分
关于树上差分
从根节点到叶子节点同一条路路径上的点是没有难度的
关键就是不同路径上的子节点怎么考虑

- 情况1：点差分（按照点来差分，边界是点，差分最后累加目的也是看点）
        power[a]++;
        power[b]++;
        power[lcaa]--;
        power[f[lcaa][0]]--;
- 情况2：线差分（解释如上）
        power[a]++;
        power[b]++;
        power[lcaa]--;

处理前需要用lca模板（构造出倍增数组）
然后再去考虑怎么差分
最后通过一个dfs从子节点回溯，计算累加和
# 动态规划

## 01背包
```
const int N = 1010;
int v[N];
int w[N];
int dp[N];
//递推前i个物品，能放入当前容量为j的最大价值（dp数组表示）
int main()
{
    int n,m;cin >> n >> m;
    for (int i = 1; i <= n; i ++ )cin >> v[i] >> w[i];
    
    for (int i = 1; i <= n;i ++){
        for(int j = m;j >= 0;j --){
            dp[j] = dp[j];
            if(j-v[i] >= 0)dp[j] = max(dp[j-v[i]] + w[i],dp[j]);
        }
    }
    cout << dp[m];
} 
```

## 完全背包
```
const int N = 1010;

int n, m;
int v[N], w[N];
int f[N];

int main()
{
    cin >> n >> m;
    for (int i = 1; i <= n; i ++ ) cin >> v[i] >> w[i];

    for (int i = 1; i <= n; i ++ )
        for (int j = v[i]; j <= m; j ++ )
            f[j] = max(f[j], f[j - v[i]] + w[i]);

    cout << f[m] << endl;

    return 0;
}
```

## 最长上升子序列O（NlogN）
```
const int N = 1e6 + 10;
int a[N],dp[N],cnt;
int main()
{
    int n;cin >> n;
    for(int i = 0;i < n;i++)scanf("%d",&a[i]);
    
    dp[cnt++] = a[0];
    for(int i = 1;i < n;i++){
        if(a[i] > dp[cnt - 1])dp[cnt++] = a[i];
        else{
            int l = 0,r = cnt - 1;
            while(l < r){//替换掉第一个大于或者等于这个数字的那个数
                int mid = (l + r)/2;
                if(dp[mid] >= a[i])r = mid;
                else l = mid + 1;
            }
            dp[r] = a[i];
        }
        //cout << cnt << endl;
    }
    cout << cnt;
}
```

## 区间dp
```
const int N = 310;

int n;
int s[N];
int f[N][N];

int main()
{
    scanf("%d", &n);
    for (int i = 1; i <= n; i ++ ) scanf("%d", &s[i]);

    for (int i = 1; i <= n; i ++ ) s[i] += s[i - 1];

    for (int len = 2; len <= n; len ++ )
        for (int i = 1; i + len - 1 <= n; i ++ )
        {
            int l = i, r = i + len - 1;
            f[l][r] = 1e8;
            for (int k = l; k < r; k ++ )
                f[l][r] = min(f[l][r], f[l][k] + f[k + 1][r] + s[r] - s[l - 1]);
        }

    printf("%d\n", f[1][n]);
    return 0;
}
```


## 数位dp的两种方式
```
/*
#include <bits/stdc++.h>
using namespace std;
using ll = long long;
ll A[22], cnt, digit, dp[22][22][2][2];
ll dfs(int pos, int cntd, bool limit, bool lead) // cntd表示目前为止已经找到多少个digit
{
    ll ans = 0;
    if (pos == cnt)
        return cntd;
    if (dp[pos][cntd][limit][lead] != -1)
        return dp[pos][cntd][limit][lead];
    for (int v = 0; v <= (limit ? A[pos] : 9); ++v)
        if (lead && v == 0)
            ans += dfs(pos + 1, cntd, limit && v == A[pos], true);
        else
            ans += dfs(pos + 1, cntd + (v == digit), limit && v == A[pos], false);
    dp[pos][cntd][limit][lead] = ans;
    return ans;
}
ll f(ll x)
{
    cnt = 0;
    memset(dp, -1, sizeof(dp));
    memset(A, 0, sizeof(A));
    while (x)
        A[cnt++] = x % 10, x /= 10;
    reverse(A, A + cnt);
    return dfs(0, 0, true, true);
}
int main()
{
    ios::sync_with_stdio(false);
    ll x, y;
    while(cin >> x >> y && x && y){
        if(x > y)swap(x,y);
        for (int i = 0; i <= 9; ++i)
        {
            digit = i;
            ll l = f(x - 1), r = f(y);
            cout << r - l << " " ;
        }
        cout << endl;
    }
    return 0;
}
*/
/*
#include <iostream>
#include <algorithm>
#include <vector>
#define debug0(x) cout << "debug0: " << x << endl
using namespace std;

const int N = 10;


// 第一种情况：
// 000~abc-1, x，999（1000个）
// 第二种情况
// abc，x，
//     1. num[i] < x, 0
//     2. num[i] == x, 0~efg
//     3. num[i] > x, 0~999
// 需要特殊讨论的情况：
//     x为 0 时，前面的部分不能全部为 0 
//     所以就要减掉一个对应10的次数


int power10(int k)
{
    int res = 1;
    while(k--)
    {
        res *= 10;
    }
    return res;
}

int get(vector<int> a,int r,int l)
{
    int res = 0;
    for(int i = r;i >= l;i --)
    {
        res = a[i] + res * 10;
    }
    return res;
}

int count(int x,int flag)
{
    if(x == 0)return 0;
    
    vector<int> a;
    while(x)
    {
        a.push_back(x%10);
        x/=10;
    }
    
    int n = a.size(),res = 0;
    for (int i = n-1 - !flag; i >= 0; i -- )//0特殊处理首位
    {
        //前缀数字长度存在
        if(i < n-1)
        {
            res += get(a,n-1,i+1)*power10(i);
            //0的情况至少要从前缀数组里减去1，因为要从末尾1开始
            if(flag == 0)res -= power10(i);
            
        }
        //前缀数组数字为最大的情况
        if(a[i] == flag)
        {
            res += get(a,i-1,0) + 1;
        }
        else if(a[i] > flag)
        {
            res += power10(i);
        }
    }
    return res;
}

int main()
{
    int a, b;
    while (cin >> a >> b , a&&b)
    {
        if (a > b) swap(a, b);

        for (int i = 0; i <= 9; i ++ )
            cout << count(b, i) - count(a - 1, i) << ' ';
        cout << endl;
    }

    return 0;
}
*/


//一般定义精度，根据题意可以适当改大或者改小，在精度要求较高的题目需要使用
//long double 的输入输出
scanf("%Lf" , &a);
printf("%.10Lf" , a);
//常用函数:fabsl(a),cosl(a).....
//即在末尾加上了字母l
//常数定义
const double eps = 1e-8;
const double PI = acos(-1.0);

int sgn(double x)//符号函数，eps使用最多的地方
{
    if (fabs(x) < eps)
        return 0;
    if (x < 0)
        return -1;
    else
        return 1;
}

//点类及其相关操作
struct Point
{
    double x, y;
    Point() {}
    Point(double _x, double _y) : x(_x), y(_y) {}
    Point operator-(const Point &b) const { return Point(x - b.x, y - b.y); }
    Point operator+(const Point &b) const { return Point(x + b.x, y + b.y); }

    double operator^(const Point &b) const { return x * b.y - y * b.x; } //叉积
    double operator*(const Point &b) const { return x * b.x + y * b.y; } //点积

    bool operator<(const Point &b) const { return x < b.x || (x == b.x && y < b.y); }
    bool operator==(const Point &b) const { return sgn(x - b.x) == 0 && sgn(y - b.y) == 0; }

    Point Rotate(double B, Point P) //绕着点P，逆时针旋转角度B(弧度)
    {
        Point tmp;
        tmp.x = (x - P.x) * cos(B) - (y - P.y) * sin(B) + P.x;
        tmp.y = (x - P.x) * sin(B) + (y - P.y) * cos(B) + P.y;
        return tmp;
    }
};

double dist(Point a, Point b) { return sqrt((a - b) * (a - b)); } //两点间距离
double len(Point a){return sqrt(a.x * a.x + a.y * a.y);}//向量的长度

struct Line
{
    Point s, e;
    Line() {}
    Line(Point _s, Point _e) : s(_s), e(_e) {}

    //两直线相交求交点
    //第一个值为0表示直线重合，为1表示平行,为2是相交
    //只有第一个值为2时，交点才有意义

    pair<int, Point> operator&(const Line &b) const
    {
        Point res = s;
        if (sgn((s - e) ^ (b.s - b.e)) == 0)
        {
            if (sgn((s - b.e) ^ (b.s - b.e)) == 0)
                return make_pair(0, res); //重合
            else
                return make_pair(1, res); //平行
        }
        double t = ((s - b.s) ^ (b.s - b.e)) / ((s - e) ^ (b.s - b.e));
        res.x += (e.x - s.x) * t;
        res.y += (e.y - s.y) * t;
        return make_pair(2, res);
    }
};

//判断线段是否相交
bool inter(Line l1, Line l2)
{
    return max(l1.s.x, l1.e.x) >= min(l2.s.x, l2.e.x) &&
            max(l2.s.x, l2.e.x) >= min(l1.s.x, l1.e.x) &&
            max(l1.s.y, l1.e.y) >= min(l2.s.y, l2.e.y) &&
            max(l2.s.y, l2.e.y) >= min(l1.s.y, l1.e.y) &&
            sgn((l2.s - l1.e) ^ (l1.s - l1.e)) * sgn((l2.e - l1.e) ^ (l1.s - l1.e)) <= 0 &&
            sgn((l1.s - l2.e) ^ (l2.s - l2.e)) * sgn((l1.e - l2.e) ^ (l2.s - l2.e)) <= 0;
}

//判断直线和线段是否相交
bool Seg_inter_line(Line l1, Line l2)
{
    return sgn((l2.s - l1.e) ^ (l1.s - l1.e)) * sgn((l2.e - l1.e) ^ (l1.s - l1.e)) <= 0;
}

//求点到直线的距离
//返回(点到直线上最近的点，垂足)
Point PointToLine(Point P, Line L)
{
    Point result;
    double t = ((P - L.s) * (L.e - L.s)) / ((L.e - L.s) * (L.e - L.s));
    result.x = L.s.x + (L.e.x - L.s.x) * t;
    result.y = L.s.y + (L.e.y - L.s.y) * t;
    return result;
}

//求点到线段的距离
//返回点到线段上最近的点
Point NearestPointToLineSeg(Point P, Line L)
{
    Point result;
    double t = ((P - L.s) * (L.e - L.s)) / ((L.e - L.s) * (L.e - L.s));
    if (t >= 0 && t <= 1)
    {
        result.x = L.s.x + (L.e.x - L.s.x) * t;
        result.y = L.s.y + (L.e.y - L.s.y) * t;
    }
    else
    {
        if (dist(P, L.s) < dist(P, L.e))
            result = L.s;
        else
            result = L.e;
    }
    return result;
}

//计算多边形面积,点的编号从0~n-1
double CalcArea(Point p[], int n)
{
    double res = 0;
    for (int i = 0; i < n; i++)
        res += (p[i] ^ p[(i + 1) % n]) / 2;
    return fabs(res);
}

//*判断点在线段上
bool OnSeg(Point P, Line L)
{
    return sgn((L.s - P) ^ (L.e - P)) == 0 &&
            sgn((P.x - L.s.x) * (P.x - L.e.x)) <= 0 &&
            sgn((P.y - L.s.y) * (P.y - L.e.y)) <= 0;
}

//求凸包Andrew算法
//p为点的编号
//n为点的数量
//ch为生成的凸包上的点
//返回凸包大小
int ConvexHull(Point *p, int n, Point *ch) //求凸包
{
    sort(p, p + n);
    n = unique(p, p + n) - p; //去重
    int m = 0;
    for (int i = 0; i < n; ++i)
    {
        while (m > 1 && sgn((ch[m - 1] - ch[m - 2]) ^ (p[i] - ch[m - 1])) <= 0)
            --m;
        ch[m++] = p[i];
    }
    int k = m;
    for (int i = n - 2; i >= 0; i--)
    {
        while (m > k && sgn((ch[m - 1] - ch[m - 2]) ^ (p[i] - ch[m - 1])) <= 0)
            --m;
        ch[m++] = p[i];
    }
    if (n > 1)
        m--;
    return m;
}

//极角排序
//叉积：对于 tmp = a x b
//如果b在a的逆时针(左边):tmp > 0
//顺时针(右边): tmp < 0
//同向: tmp = 0
//相对于原点的极角排序
//如果是相对于某一点x,只需要把x当作原点即可
bool mycmp(Point a, Point b)
{
    if (atan2(a.y, a.x) != atan2(b.y, b.x))
        return atan2(a.y, a.x) < atan2(b.y, b.x);
    else
        return a.x < b.x;
}

//判断点在凸多边形内
//要求
//点形成一个凸包，而且按逆时针排序
//如果是顺时针把里面的<0改为>0
//点的编号:0~n-1
//返回值：
//-1:点在凸多边形外
//0:点在凸多边形边界上
//1:点在凸多边形内
int inConvexPoly(Point a, Point p[], int n)
{
    for (int i = 0; i < n; i++)
    {
        if (sgn((p[i] - a) ^ (p[(i + 1) % n] - a)) < 0)
            return -1;
        else if (OnSeg(a, Line(p[i], p[(i + 1) % n])))
            return 0;
    }
    return 1;
}

//判断点是否在凸包内
bool inConvex(Point A, Point *p, int tot)
{
    int l = 1, r = tot - 2, mid;
    while (l <= r)
    {
        mid = (l + r) >> 1;
        double a1 = (p[mid] - p[0]) ^ (A - p[0]);
        double a2 = (p[mid + 1] - p[0]) ^ (A - p[0]);
        if (a1 >= 0 && a2 <= 0)
        {
            if (((p[mid + 1] - p[mid]) ^ (A - p[mid])) >= 0)
                return true;
            return false;
        }
        else if (a1 < 0)
            r = mid - 1;
        else
            l = mid + 1;
    }
    return false;
}

//判断点在任意多边形内
//射线法，poly[]的顶点数要大于等于3,点的编号0~n-1
//返回值
//-1:点在凸多边形外
//0:点在凸多边形边界上
//1:点在凸多边形内
int inPoly(Point p, Point poly[], int n)
{
    int cnt;
    Line ray, side;
    cnt = 0;
    ray.s = p;
    ray.e.y = p.y;
    ray.e.x = -100000000000.0; //-INF,注意取值防止越界

    for (int i = 0; i < n; i++)
    {
        side.s = poly[i];
        side.e = poly[(i + 1) % n];

        if (OnSeg(p, side))
            return 0;

        //如果平行轴则不考虑
        if (sgn(side.s.y - side.e.y) == 0)
            continue;

        if (OnSeg(side.s, ray))
        {
            if (sgn(side.s.y - side.e.y) > 0)
                cnt++;
        }
        else if (OnSeg(side.e, ray))
        {
            if (sgn(side.e.y - side.s.y) > 0)
                cnt++;
        }
        else if (inter(ray, side))
            cnt++;
    }
    if (cnt % 2 == 1)
        return 1;
    else
        return -1;
}

//判断凸多边形
//允许共线边
//点可以是顺时针给出也可以是逆时针给出
//但是乱序无效
//点的编号0，n-1
bool isconvex(Point poly[], int n)
{
    bool s[3];
    memset(s, false, sizeof(s));
    for (int i = 0; i < n; i++)
    {
        s[sgn((poly[(i + 1) % n] - poly[i]) ^ (poly[(i + 2) % n] - poly[i])) + 1] = true;
        if (s[0] && s[2])
            return false;
    }
    return true;
}

//判断凸包是否相离
//凸包a：n个点,凸包b：m个点
//凸包上的点不能出现在另一个凸包内
//凸包上的线段两两不能相交
bool isConvexHullSeparate(int n, int m, Point a[], Point b[])
{
    for (int i = 0; i < n; i++)
        if (inPoly(a[i], b, m) != -1)
            return false;

    for (int i = 0; i < m; i++)
        if (inPoly(b[i], a, n) != -1)
            return false;

    for (int i = 0; i < n; i++)
    {
        for (int j = 0; j < m; j++)
        {
            Line l1 = Line(a[i], a[(i + 1) % n]);
            Line l2 = Line(b[j], b[(j + 1) % m]);
            if (inter(l1, l2))
                return false;
        }
    }
    return true;
}

*/

```
