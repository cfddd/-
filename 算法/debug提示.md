## 2023.5.26

![img](https://img2023.cnblogs.com/blog/2740326/202305/2740326-20230526222407099-1930763702.png)
![img](https://img2023.cnblogs.com/blog/2740326/202305/2740326-20230526222345197-1356287867.png)

- <https://codeforces.com/contest/1837/my>
- 如图，当把所有的逻辑都做完之后，错在了在线上，直接return了，G
- 一定要有良好的逻辑分支，做到一丝不苟！
- 当出现bug时，只有两种情况，要么是别人错了，要么是我的代码某个地方错了，这个地方，一定可能是任何地方！

## 2023.4.27

- 又是pow的精度损失，无语，cf没把我卡掉
- <https://codeforces.com/contest/1444/submission/203241425>
![img](https://img2023.cnblogs.com/blog/2740326/202304/2740326-20230427211053070-666571351.png)

## 2023.4.25

- <https://codeforces.com/contest/1822/my>
- 卡常教训：把所有的内存占用都移出栈区，数据量很大的时候，在栈区里面开空间非常耗费时间！
- 尽量少使用vector当成数组！！！
- 卡常很极限可以用map和set等stl在特殊的地方优化

## 2023.4.2

- [还是数学有关的](https://atcoder.jp/contests/abc296/tasks/abc296_d)
- 会超出LL，需要做一些特殊处理
![img](https://img2023.cnblogs.com/blog/2740326/202304/2740326-20230402150324889-243999710.jpg)
- 如图，sqrt在对64位整数运算时会产生极大的误差，必须再手动校准(sqrtl也没有好到哪里去)
- 找到第一个小于等于$m$开平方数字

```
int limit = sqrtl(m);
    while((limit - 1)*(limit - 1) >= m)
        limit--;
    while ((limit + 1) * (limit + 1) < m)
        limit++;
```

- 找到第一个大于等于$m$开平方数字

```
    LL x = sqrtl(m);
    for(;x*x>m;x--){}
    for(;x*x<=m;x++){}
```

- 还有很重要的一点，cout会自动把double转为科学计数法，这必须要注意！！！

## 2023.3.24

- 炸裂的精度ceil，long long类型数字开平方sqrt有非常多的精度损失
- 导致ceil的准确率非常低
- 所以开平方禁止使用sqrt，尤其是在此情况下

`debug2(ceil(2 + eps),2+eps)`
double只有8字节，相当于只能存储20位整数，包括小数和整数部分的位数和，所以这样后的表达式double类型为2，精度损失
输出为`ceil(2 + eps) = 2  2+eps = 2`

- [相当惨烈的结果，死在了基础上](https://codeforces.com/contest/1809/problem/B)

## 2023.3.11

- 不要使用INT_MAX等宏定义做变量，很容易产生越界
- lower_bound和upper_bound都会在set.end()越界，必须特判
- 分类讨论太多的时候必须非常细致
- 边界问题，尽量不要用特殊值替代（除非很熟练），做到能用自然语言描述就好
- <https://codeforces.com/contest/1802/my>

## 2023.2.27

- debug就是逐步缩小范围，甚至是一句一句调试

## 2023.2.14

- 下标就是折磨人，要找能避开或者减少下标的方法
- <https://codeforces.com/contest/1374/my>

## 2023.1.30

- <https://atcoder.jp/contests/abc270/submissions/me>
- 写代码必须注意细节，注意提高代码的撸棒性
- 一直交代码不太现实
- <https://ac.nowcoder.com/acm/contest/view-submission?submissionId=60669813&returnHomeType=1&uid=808375229>
- ide里面一个普通的 1 是 int 类型，左位移60位找死，用1LL

## 2023.1.19

- <https://ac.nowcoder.com/acm/contest/view-submission?submissionId=60505927>
- 问题：当T测试样例数量较大时，尽量不要使用memset，耗时非常严重
- 解决办法：修改一些特殊点，或者优化代码往其他方向
