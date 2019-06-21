---
title: git merge origin branch和git merge origin/branch的区别
date: 2019-08-10 13:12:01
tags:
    - git
categories:
    - [git, 实验]
---

## 现象
某git仓库的版本历史如下
![](https://raw.githubusercontent.com/MilesGO517/images/master/2019/8/6.png)
```
git checkout dev  # git会自动帮你创建本地dev分支，跟踪origin/dev
git rebase -i HEAD~3  # 默认情况下忽略merge 节点
```
rebase设置为pick、drop、pick，解决冲突并将rebase完成后，版本历史如下
![](https://raw.githubusercontent.com/MilesGO517/images/master/2019/8/7.png)
```
git merge origin dev
```
解决冲突后，产生了如下的版本历史
![](https://raw.githubusercontent.com/MilesGO517/images/master/2019/8/8.png)
新的dev节点，竟然不是origin/dev的后代？

若使用`git merge origin/dev`，则结果就是正常的

<!--more-->

## 分析
`git merge origin dev`这种写法，在语义上应理解为，git merge origin & git merge dev。由于当前正在dev分支下，后者不起作用，那前者又是什么意思呢？
观察版本历史，dev的其中一个父节点，包含三个引用，`origin/master`、`origin/HEAD`、`master`，估计origin会被解释为`origin/master`或者`origin/HEAD`

## 结论
```
git merge origin dev = git merge origin & git merge dev
git merge origin = git merge origin/HEAD
```
查看一下origin/HEAD中的内容
```
cat .git/refs/remotes/origin/HEAD
显示结果ref: refs/remotes/origin/master
```
所以origin/HEAD指向origin/master，`git merge origin dev`最终产生的效果，等价于`git merge origin/master`

## 还没完
### git remote set-head
https://linux.die.net/man/1/git-remote
```
set-head
Sets or deletes the default branch ($GIT_DIR/remotes/<name>/HEAD) for the named remote. Having a default branch for a remote is not required, but allows the name of the remote to be specified in lieu of a specific branch. For example, if the default branch for origin is set to master, then origin may be specified wherever you would normally specify origin/master.
```
所以说，origin/HEAD在语义上，指的是远端仓库的默认分支，假设为dev
- 假设远端仓库托管在github上，则进入该仓库页面，显示的内容就是dev分支的内容
- git clone该仓库到本地后，首次进入该目录，就会在dev分支下
- git clone该仓库到本地后，查看origin/HEAD，指向的应该是origin/dev

### How does origin/HEAD get set?
https://stackoverflow.com/questions/8839958/how-does-origin-head-get-set
- 修改local copy
    - 直接修改.git/refs/remotes/origin/HEAD
    - git remote set-head origin some_branch
- 真正修改远端仓库（以github为例）
    - Some repository-->Settings--->Branches--->Default Branch




