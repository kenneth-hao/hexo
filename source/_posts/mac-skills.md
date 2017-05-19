title: Mac Skills - Improve Efficiency (Undone)
date: 2017-04-24 22:00:00
tags: [mac, skill]

---

## Tools

### Homebrew - 软件包管理工具

类似 Ubuntu 下的 `apt-get` .

使用 Homebrew 可以安装 macOS 没有预装 但你需要的东西, 比如 `wget`

```shell
brew install wget
brew install nginx
```

也可以用来安装 macOS 预装了，但版本较低的软件，比如 `vim`

```shell
brew install vim
```

软件包存放目录 `/usr/local/Cellar`

### Homebrew-Cask - Mac Application 的命令行管理工具

> “To install, drag this icon…” no more!

它可以让我们更加优雅、简洁、快速的管理我们的 Mac 应用。

安装:

```shell
brew tap caskroom/cask
```

正常的 Mac 软件安装流程：打开官网，下载，双击 `*.dmg` , `drag this icon into Folder Application`

更加优雅的安装方式：

```shell
brew cask install google-chrome
```

软件存放目录 `/usr/local/Caskroom`

推荐安装的软件

```shell
# Chrome 浏览器
brew cask install google-chrome
# 文件同步工具 (需翻墙)
brew cask install dropbox
# 文本编辑器
brew cask install sublime-text
# Markdown 编辑器
brew cask install typora
```

<!-- more -->

### iTerm2



### Zsh



### Oh My Zsh



iTerm2

- Ctrl + R: 反向检索 history，例如: 检索登录过的服务器，快速登录服务器



autojump

paste 工具： 记录 paste 过的内容

Alfred

Dropbox 