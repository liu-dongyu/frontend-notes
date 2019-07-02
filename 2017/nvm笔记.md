> https://segmentfault.com/a/1190000004404505

### node 版本管理工具

### 安装

```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.30.2/install.sh | bash
```

or

```
git clone https://github.com/creationix/nvm.git ~/.nvm
source ~/.nvm/nvm.sh
```

### 相关指令

```
# 查看已安装的版本
nvm ls

# 查看可以安装的版本
nvm ls-remote

# 安装指定的版本
nvm install <version>

# 删除指定的版本
nvm uninstall <version>

# 使用选定的版本
nvm use <version>

# 选定默认版本
nvm alias default <version>
```
