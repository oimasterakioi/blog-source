---
title: 在 Ubuntu 上安装 node.js
date: 2022-06-21 14:04:55
tags: Ubuntu, nodejs
---

最近要在 Ubuntu 上搭一个 node.js 的运行环境，但是因为自带的软件源中 node.js 的版本实在太老，于是我决定使用工具 nvm 进行安装。

nvm 全称 Node Version Manager，可以帮助我们快速管理 node.js 的版本。为了下载这个工具，需要执行这个命令。

```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

为了以防万一，我们需要执行以下命令来适应更改。

```sh
source ~/.bashrc
```

接下来，就可以看看有哪些版本可以被安装。请运行下面的脚本。它将会输出一个很长的列表，不要着急。然后就可以选择了。

```sh
nvm ls-remote
```

当然，列表中也会有一些注释，例如哪个版本是最新的 LTS。

最后输入安装。下面是我使用的版本，具体地还是需要看需求。

```sh
nvm install v16.15.1
```

如果你安装了多个版本的话，也可以通过 `use` 命令来进行切换。

```sh
nvm use v16.15.1
```

一般来说，nvm 会自动切换到最新安装的一个版本。不过为了以防万一，不如随手运行一下！

如果需要看看那些版本被安装，运行下面脚本。

```sh
nvm ls
```