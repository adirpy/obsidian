
## 直接安装

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## 卸载

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/uninstall.sh)"
```

## 替换brew\.git:

```bash
 cd "$(brew --repo)"
 # 中国科大:
 git remote set-url origin https://mirrors.ustc.edu.cn/brew.git
 # 清华大学:
 git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git
```

## 替换homebrew-core.git:

```bash
 cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
 # 中国科大:
 git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
 # 清华大学:
 git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
```

## 替换homebrew-bottles:

```bash
 # 中国科大:
 echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.ustc.edu.cn/homebrew-bottles' >> ~/.bash_profile
 source ~/.bash_profile
 # 清华大学:
 echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.bash_profile
 source ~/.bash_profile
```

## 应用生效:

`brew update`

## 常装软件

```bash
brew cask install qlcolorcode qlstephen qlmarkdown quicklook-json qlprettypatch quicklook-csv qlimagesize  suspicious-package quicklookase qlvideo Itsycal iina postman
```

