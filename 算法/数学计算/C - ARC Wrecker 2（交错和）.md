## 题目
- https://atcoder.jp/contests/arc119/tasks/arc119_c
## 思路
- 交错和
## 代码
```
onst int N = 3e5+10;
int v[N];
void solve()
{
    int n;cin >> n;
	for (int i = 1; i<=n; i++){
        cin >> v[i];
		if (i%2 == 0){
			v[i]*=-1;
		}
	}
    map<long long, long long> mp;
 
	mp[0] = 1;
 
	long long ans = 0;
	long long suma = 0;
 
	for (int i = 1; i<=n; i++){
		suma+=v[i];
		ans+=mp[suma];
		mp[suma]++;
	}
 
	cout << ans;
}
```