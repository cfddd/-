[官方文档](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%89%93%E6%A0%87%E7%AD%BE)
## 标签介绍
像其他版本控制系统（VCS）一样，Git 可以给仓库历史中的某一个提交打上标签，以示重要。
## 列出标签
在 Git 中列出已有的标签非常简单（可带上可选的 `-l` 选项 `--list`）：
```git
$ git tag
v1.0
v1.0-lw

$ git tag -l "v1.0*"
v1.0
v1.0-lw
```
## 创建标签

Git 支持两种标签：轻量标签（lightweight）与附注标签（annotated）

### 附注标签
附注标签是存储在 Git 数据库中的一个完整对象， 它们是可以被校验的，其中包含：
- 打标签者的名字
- 电子邮件地址
- 日期时间，
- 标签信息，可以使用 GNU Privacy Guard （GPG）签名并验证

通常会建议创建附注标签，这样你可以拥有以上所有信息。但是如果你只是想用一个临时的标签， 或者因为某些原因不想要保存这些信息，那么也可以用轻量标签。
```git
 git tag -a v1.0 -m "my version 1.0"
```
### 辅助标签
轻量标签很像一个不会改变的分支——它只是某个特定提交的引用。
```git
git tag v1.0-lw
```
### 后期打标签
上面的命令都会对当前提交打标签，但是有的时候需要给以前的提交打标签

**先列出git提交信息**
```git
git log --pretty=oneline
```

**假设现在有很多提交**
```shell
c4....... (HEAD -> master, tag: v1.0-lw, tag: v1.0) fix:格式问题
aa....... 作业4
```

**选择`aa`进行打标签**
```git
 git tag -a v0.1 aa8d -m "good"
```
**结果验证**
```shell
git log --pretty=oneline
c4...... (HEAD -> master, tag: v1.0-lw, tag: v1.0) fix:格式问题
aa...... (tag: v0.1) 作业4
```
## 查看标签
```git
git show v1.0
```
## 共享标签
**默认情况下，`git push` 命令并不会传送标签到远程仓库服务器上。** 在创建完标签后你必须显式地推送标签到共享服务器上。

这个过程就像共享远程分支一样——你可以运行 `git push origin <tagname>`

```git
git push origin v1.0
```

**上传很多标签**

这将会把所有不在远程仓库服务器上的标签全部传送到那里。
```git
git push origin --tags
```
## 删除标签
```git
git tag -d v1.0-lw
```
上述命令并不会从任何远程仓库中移除这个标签，你必须用 `git push <remote> :refs/tags/<tagname>` 来更新你的远程仓库：

```git
git push origin :refs/tags/v1.0-lw
```
第二种更直观的删除远程标签的方式是：

```git
$ git push origin --delete <tagname>
```