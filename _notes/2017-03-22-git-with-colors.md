---
title: "git命令内容彩色输出"
tags:
  - git
  - config
---


最近需要把工作环境从笔记本切换到集群上，
所以重新在上面建了`zsh` + `tmux` + `git`的环境。
但奇怪的是，一样的配置过程，一样的配置文件，
在自己的笔记本上git的所有命令都能够彩色现实(**这个很重要**),
但在集群上，却只有单调的灰白色。

查询了一番，发现可以通过`git config`来进行配置，

``` shell
git config --global color.ui auto
git config --global color.branch auto
git config --global color.status auto
```

或者直接在`~/.gitconfig`里面写配置文件也行：

``` ini
[color]
  diff = auto
  status = auto
  branch = auto
  interactive = auto
  ui = true
  pager = true

# 或者更精细的控制
[color "status"]
  added = green
  changed = red bold
  untracked = magenta bold

[color "branch"]
  remote = yellow
```
