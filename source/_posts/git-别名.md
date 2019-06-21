---
title: git-别名
date: 2019-08-11 14:54:51
tags:
    - git
categories:
    - [git, 最佳实践]
---

https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-Git-%E5%88%AB%E5%90%8D
git不会在你输入部分命令时自动推断出你想要的命令。可以通过别名来简化命令。

## 设置方法
```
git config --global alias.别名 'git原指令名'
git config --global alias.别名 '!外部原指令名'
```

## 我的别名
```
git config --global alias.lol "log --oneline --graph --decorate --all"
git config --global alias.bvv "branch -vv"
git config --global alias.bav "branch -av"
git config --global alias.cm "commit -m"
git config --global alias.cam "commit -am"
git config --global alias.camd "commit --amend"
```