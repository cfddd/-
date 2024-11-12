> 来源：
> [F04 扩展 KMP(Z 函数)_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Y54y1o7Ca/?spm_id_from=333.337.search-card.all.click&vd_source=2d885cb62bb9393fa8a5379c72eabd82)
> [Z 函数（扩展 KMP） - OI Wiki (oi-wiki.org)](https://oi-wiki.org/string/z-func/)
![](http://file.cfd.hhblog.top/myPicture/20240226224927.png)

![](http://file.cfd.hhblog.top/myPicture/20240226224950.png)

```c++
vector<int> z_function(string s) {
  int n = (int)s.length();
  s = "#" + s;
  vector<int> z(n+1,0);
  z[1] = n;
  for (int i = 2, l = 0, r = 0; i <= n; ++i) {
    if(i <= r)z[i] = min(z[i - l + 1],r - i + 1);
    while(s[1 + z[i]] == s[i + z[i]])z[i] ++;
    if(i + z[i] - 1 > r)l = i,r = i + z[i] - 1;
  }
  return z;
}
```
