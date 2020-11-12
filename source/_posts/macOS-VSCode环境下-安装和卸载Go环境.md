---
title: 'macOS+VSCode环境下,安装和卸载Go环境'
tags:
  - Go
categories:
  - 开发环境
date: 2020-11-12 16:43:06
---


> 最近在查看后台 API 文档，在自己电脑上需要搭建 Go 环境。网上很多基于 macOS、VSCode 的环境搭建教程都落后了，不够与时俱进，顾将这次 macOS + VSCode + Go 环境搭建记录下来。

本文介绍以下几个问题：

1. Go 环境的安装
2. VSCode 集成 Go 开发环境及插件
3. Go 环境的卸载

## Go 环境的安装

> Go 安装可以通过图形可视化安装[官网下载链接](https://golang.org/dl/)。本文是基于 macOS，通过 HomeBrew，命令行安装更简便，后续升级维护简便。

直接执行以下命令来安装Go：

```bash
brew install go
```

此时可能会卡在 Updating Homebrew，可以通过切换中科大源来解决：

```bash
// 替换brew.git:
cd "$(brew --repo)"
git remote set-url origin https://mirrors.ustc.edu.cn/brew.git

// 替换homebrew-core.git:
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
```

然后就可以顺利安装了。安装完成后，可以在终端执行以下命令来测试：

```bash
go version
```

此时会在终端显示go版本：

![img](/uploads/image-20201112162352406.png)

### Go环境变量配置

在 Go 1.13 之后，无需再通过设置系统环境变量的方式来修改，可以通过 go env -w 命令来设置 Go 的环境变量。
需要设置的环境变量如下：

```javascript
// 用于存放依赖包以及编译文件，比较随意，只要不和GOROOT重名即可，官方禁止这一行为
go env -w GOPATH=/Users/username/go
// 设置代理后，在未翻墙的情况下，打开vscode后gopls工具的加载会很快
go env -w GOPROXY=https://goproxy.cn,direct
```

## VSCode 安装 Go 插件

在插件商店中搜索 Go(Publish By Go Team at Google)，并点击安装即可。

> 插件安装好后，用 VSCode 打开一个 .go 的文件，会提示安装 Go 相关插件，根据个人需求点击提示上的 “Install” 或者 “Install All”，稍等片刻会看到插件都已经安装 SUCCSESS。

## Go 环境的卸载

1. 删除 /usr/local 下的 go 目录。

   > 备注：这个目录是安装 go 的时候自动生成的。 如果删除完，终端中使用 go version 命令，会报找不到 go 命令。

2. 删除 Path 环境变量。

   > 备注：这里，我只是想删除一个 Go 版本， 所以， GOPATH 还是需要的。

3. 删除配置文件信息：在 /etc/profile 或者 \$HOME/.profile 或者 \$HOME/.bahs_profile 中删除 bin 的设置。

4. 删除 macOS 的安装包文件，在 /etc/paths.d/go 目录下。

   > 我只用了第一步，重新安装，其他都还继续使用。
