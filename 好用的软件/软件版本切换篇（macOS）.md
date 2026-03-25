使用软件管理工具brew安装的，可以参考本目录快速切换软件版本
## jdk
核心命令：/usr/libexec/java_home
macOS 提供的工具，用于查找和管理已安装的 Java 版本。

下面是 setjdk() 函数（在 ~/.zprofile 中）
```sh
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-11.jdk/Contents/Home

# 定义一个函数来切换JDK版本
function setjdk() {
    if [ $# -ne 0 ]; then
        removeFromPath '/System/Library/Frameworks/JavaVM.framework/Home/bin'
        removeFromPath '/Library/Java/JavaVirtualMachines/*/Contents/Home/bin'
        if [ -n "${JAVA_HOME+x}" ]; then
            removeFromPath $JAVA_HOME
        fi
        export JAVA_HOME=`/usr/libexec/java_home -v $@`
        export PATH=$JAVA_HOME/bin:$PATH
    fi
}

# 删除 PATH 中的旧 JDK 路径
function removeFromPath() {
    export PATH=$(echo $PATH | sed -E -e "s;:$1;;" -e "s;$1:?;;")
}
```

工作原理：
清理旧路径：从 PATH 中移除旧的 Java 路径
查找新版本：使用 /usr/libexec/java_home -v <版本号> 获取指定版本的路径
设置环境变量：
JAVA_HOME：Java 安装目录
PATH：将新版本的 bin 目录添加到 PATH 前面

### 使用方法
```sh
# 切换到 Java 17
setjdk 17

# 切换到 Java 21
setjdk 21

# 切换到 Java 8
setjdk 1.8
```

## npm node版本

### 1. 配置加载（在 `~/.zshrc` 中）

```6:7:~/.zshrc
export NVM_DIR="$HOME/.nvm"
  [ -s "/opt/homebrew/opt/nvm/nvm.sh" ] && \. "/opt/homebrew/opt/nvm/nvm.sh"  # This loads nvm
```

- 设置 `NVM_DIR` 环境变量（默认 `~/.nvm`）
- 加载 nvm 脚本（通过 Homebrew 安装）

### 2. 核心机制

#### 版本存储位置
- 所有 Node.js 版本安装在：`~/.nvm/versions/node/`
- 例如：`~/.nvm/versions/node/v18.20.8/`

#### 切换原理（`nvm use` 命令）

当你执行 `nvm use 18` 时，nvm 会：

1. 修改 PATH 环境变量：
   ```bash
   # 将当前版本的 bin 目录添加到 PATH 最前面
   PATH="~/.nvm/versions/node/v18.20.8/bin:$PATH"
   ```

2. 设置环境变量：
   ```bash
   export NVM_BIN="~/.nvm/versions/node/v18.20.8/bin"
   export NVM_INC="~/.nvm/versions/node/v18.20.8/include/node"
   ```

3. 创建符号链接（可选）：
   ```bash
   # 如果启用了 NVM_SYMLINK_CURRENT
   ln -s ~/.nvm/versions/node/v18.20.8 ~/.nvm/current
   ```

### 3. 别名系统

- 别名存储在：`~/.nvm/alias/`
- 默认别名：`~/.nvm/alias/default` → 指向版本号（如 `18`）
- 自定义别名：可以创建任意别名文件

### 4. 常用命令

```bash
# 列出已安装的版本
nvm ls

# 切换到指定版本
nvm use 18

# 安装新版本
nvm install 20

# 设置默认版本（新 shell 自动使用）
nvm alias default 18

# 查看当前版本
nvm current

# 列出远程可用版本
nvm ls-remote
```
## maven 服务器切换
保留三份xml配置即可
```sh
#!/bin/bash

# Maven 配置目录
MAVEN_DIR="$HOME/.m2"

# 主配置文件
TARGET="$MAVEN_DIR/settings.xml"

case $1 in
  aliyun)
    cp "$MAVEN_DIR/settings.aliyun.xml" "$TARGET"
    echo "✅ 已切换到 Aliyun Maven"
    ;;
  lkcoffee)
    cp "$MAVEN_DIR/settings.lkcoffee.xml" "$TARGET"
    echo "✅ 已切换到 lkcoffee Nexus"
    ;;
  central)
    # 假设 settings.xml.bak 是最初始 central 配置
    cp "$MAVEN_DIR/settings.xml.bak" "$TARGET"
    echo "✅ 已切换到 Maven Central"
    ;;
  *)
    echo "用法: $0 [aliyun|lkcoffee|central]"
    ;;
esac

```

执行命令
```sh
/Users/luckincoffee/.m2/switch-maven-mirror.sh aliyun
```


## 切换 Python 版本的方法

### 1. **项目级别切换（推荐）**
在项目目录中设置本地 Python 版本，会创建 `.python-version` 文件：

```bash
# 切换到 Python 3.10.13
pyenv local 3.10.13

# 切换到 Python 3.12.8
pyenv local 3.12.8

# 切换到系统 Python
pyenv local system

# 取消项目本地设置（删除 .python-version）
pyenv local --unset
```

### 2. **全局切换**
设置全局默认 Python 版本（影响所有项目）：

```bash
# 设置全局 Python 版本
pyenv global 3.10.13

# 查看全局版本
pyenv global
```

### 3. **临时切换（仅当前 shell 会话）**
仅对当前终端会话生效：

```bash
# 临时使用某个版本
pyenv shell 3.10.13

# 取消临时设置
pyenv shell --unset
```

### 4. **查看可用版本**
```bash
# 查看已安装的所有版本
pyenv versions

# 查看所有可安装的版本
pyenv install --list | grep "3.10"
```

### 5. **安装新版本**
如果需要安装其他 Python 版本：

```bash
# 安装 Python 3.11.x
pyenv install 3.11.9

# 安装后即可切换使用
pyenv local 3.11.9
```
