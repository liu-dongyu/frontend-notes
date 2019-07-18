> 不翻墙寸步难行

## 安装过程

### 长时间`Checking Android licenses is taking an unexpectedly long time`

尝试运行`flutter doctor --android-licenses`

###  长时间 Updating Homebrew...

1. 换源

```zsh
# .zshrc

# brew中科大源
brew_ustc () {
  cd "$(brew --repo)"
  git remote set-url origin https://mirrors.ustc.edu.cn/brew.git
  cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
  git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
  cd ~/
}

# brew官方源
brew_reset () {
  cd "$(brew --repo)"
  git remote set-url origin https://github.com/Homebrew/brew.git
  cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
  git remote set-url origin https://github.com/Homebrew/homebrew-core.git
  cd ~/
}
```

```bash
source ./zshrc
brew_ustc # 切换到中科大源
brew_reset # 切换回官方源
```

2. 如果有 v2ray 等翻墙手段，可让 zsh 走代理

```zsh
# .zshrc
proxy () {
  export ALL_PROXY=socks5://127.0.0.1:1081
  echo "HTTP Proxy on"
}
proxy_off () {
  unset ALL_PROXY
  echo "HTTP Proxy off"
}
alias ip="curl cip.cc"
```

```bash
proxy # 开代理
proxy_off # 关代理
ip # 查看出口IP是否翻墙服务器
```

### 长时间 Initializing gradle

本地没有 gradle，程序需要去官网下载。要么开代理，要么手动下载

待续......
