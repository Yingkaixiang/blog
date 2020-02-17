更换镜像源

```zsh
cd "$(brew --repo)"

# 中科大
git remote set-url origin https://mirrors.ustc.edu.cn/brew.git

# 官方
git remote set-url origin https://github.com/Homebrew/brew.git

# 清华
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git
```

实用命令

brew list <应用名称> 查看应用的安装位置

homebrew-bottles 镜像源

```zsh
# 临时替换
export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles
```

```zsh
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.bash_profile
source ~/.bash_profile
```

https://www.raydbg.com/2019/Homebrew-Update-Slow/