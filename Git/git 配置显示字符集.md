```sh
$ git status --long
On branch master
Your branch is up to date with 'note/master'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   "star \346\263\225\345\210\231.md"
        new file:   "\345\260\261\344\270\232\344\271\213\346\230\237\344\273\213\347\273\215.md"

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   "\345\260\261\344\270\232\344\271\213\346\230\237\344\273\213\347\273\215.md"

```

建议同时设置 `core.quotepath` 和 `LANG` 环境变量，这样可以确保 Git 在各种情况下都能正确显示中文路径：
```bash
git config --global core.quotepath false
echo 'export LANG=en_US.UTF-8' >> ~/.bashrc  # 或 ~/.zshrc
```

设置完成后，再次运行 `git status` 应该能正确显示中文文件名。

```sh
$ git status
On branch master
Your branch is up to date with 'note/master'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   star 法则.md
        new file:   就业之星介绍.md

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   就业之星介绍.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        Git/未命名.md
```
