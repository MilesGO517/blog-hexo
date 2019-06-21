---
title: '【git】- git branch'
date: 2019-08-09 15:43:00
tags:
    - git
categories:
    - [git, 指令介绍]
---

## 常用
- `git branch -av`
- `git branch -vv`

<!--more-->

## git文档
```
git branch
-v
-vv
--verbose
When in list mode, show sha1 and commit subject line for each head, along with relationship to upstream branch (if any). If given twice, print the name of the upstream branch, as well (see also git remote show <remote>).
```

## 翻译说明
显示每个（本地）分支当前指向的提交记录的哈希值，以及和其上游分支的相对位置（如果有的话）
`-v`与`-verbose`是一个效果
`-vv`会显示上游分支的名字

## 举例
```
$ git branch -vv
  cyr_branch   689c970 [origin/cyr_branch: behind 2] default
* master       f1e2810 finally
  test-mysql   e2fa7b9 add .gitignore
  test-network 0d426ce add .gitignore
  xu_branch    e55fcf1 [origin/xu_branch: ahead 1] debug
  zfc_branch   e4a40a5 [origin/zfc_branch] merge
```
`cyr_branch`的上游分支为`origin/cyr_branch`，前者落后后者2
`xu_branch`的上游分支为`origin/xu_branch`，前者领先后者1
`zfc_branch`的上游分支为`origin/zfc_branch`，两者同步
`test-mysql`和`test-network`没有对应的上游分支
`master`的上游分支为`origin/master`，但是没有显示，我猜测是因为这个对应关系是固定的，所以无需显示

## 组合用法
### -a 显示所有本地及远端分支名
```
$ git branch -a
  cyr_branch
* master
  test-mysql
  test-network
  xu_branch
  zfc_branch
  remotes/origin/cyr_branch
  remotes/origin/master
  remotes/origin/xu_branch
  remotes/origin/zfc_branch
```
### -av
```
$ git branch -av
  cyr_branch                689c970 [behind 2] default
* master                    f1e2810 finally
  test-mysql                e2fa7b9 add .gitignore
  test-network              0d426ce add .gitignore
  xu_branch                 e55fcf1 [ahead 1] debug
  zfc_branch                e4a40a5 merge
  remotes/origin/cyr_branch 1e63522 cyr
  remotes/origin/master     f1e2810 finally
  remotes/origin/xu_branch  1c82da3 adjust db
  remotes/origin/zfc_branch e4a40a5 merge
```

### -avv
```
$ git branch -avv
  cyr_branch                689c970 [origin/cyr_branch: behind 2] default
* master                    f1e2810 finally
  test-mysql                e2fa7b9 add .gitignore
  test-network              0d426ce add .gitignore
  xu_branch                 e55fcf1 [origin/xu_branch: ahead 1] debug
  zfc_branch                e4a40a5 [origin/zfc_branch] merge
  remotes/origin/cyr_branch 1e63522 cyr
  remotes/origin/master     f1e2810 finally
  remotes/origin/xu_branch  1c82da3 adjust db
  remotes/origin/zfc_branch e4a40a5 merge
```