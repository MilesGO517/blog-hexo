---
title: git-push、git-pull、git-fetch、git-merge、git-rebase的参数问题
date: 2019-08-10 13:52:19
tags:
    - git
categories:
    - [git, 实验]
    - [git, 最佳实践]
---
## 写在前面
- 这几个指令，都涉及到了远端仓库、远端分支
- 需要研究一下这几个指令分别以怎样的形式指定这两个参数
- 后文也仅讨论这两个参数

## 使用建议
- git pull
    - 无论哪种情况，使用前先checkout到需要merge的本地分支
    - 跟踪远程同名分支，然后git pull不带参数执行即可
    - 可以指定远程仓库名和分支名，无视跟踪关系，无视是否同名，merge到当前的本地分支
- git push
    - 跟踪远程同名分支，checkout到这个分支，然后git push不带参数执行即可
    - 可以指定远程仓库名和分支名，无视跟踪关系，无视当前本地所在分支，将本地同名分支push远端
- git fetch
    - 没什么特别的
    - 请看《总的结论》
- git merge
    - 没什么特别的
    - 请看《总的结论》

<!--more-->

## 总的结论
- git pull
    - 不带参数
        - 没有跟踪分支，则放出提示
        - 有跟踪分支，且同名，则成功（常见情境）
        - 有跟踪分支，且不同名，则成功
    - 只带远端仓库名
        - 不会展开为<远端仓库名>/HEAD
        - 没有跟踪分支，则放出提示
        - 有跟踪分支，但是跟踪分支不属于指定的远端仓库名，则放出提示
        - 有跟踪分支，且属于指定的远端仓库名，则成功
    - 带远端仓库名和远端分支名（常见情境）
        - 两个参数与空格分隔
        - 无视跟踪关系
        - 将指定的远端分支merge到当前本地分支
- git push
    - 不带参数
        - 没有跟踪分支，则提示放出提示
        - 有跟踪分支，且同名，则成功（常见情境）
        - 有跟踪分支，且不同名，则放出提示
    - 只带远端仓库名
        - 不会展开为<远端仓库名>/HEAD
        - 实验结果有点奇怪，不建议这种用法
    - 带远端仓库名和远端分支名（常见情境）
        - 两个参数以空格分隔
        - 无视当前所在分支名
        - 无视跟踪关系
        - 将本地的同名分支push到远端
    - `git push <远端仓库名> <本地引用>:<远端分支名>` 
- git fetch
    - 可以指定远程仓库名、远程分支名，空格分隔
    - 可以只指定远程仓库名
    - 可以使用--all参数，fetch所有远程仓库
    - 可以将远程仓库分组，进行分组fetch
    - 可以使用--multiple参数，fetch多个远程仓库和分组

<!--more-->
## git-pull
### 指令格式
`git pull [<options>] [<repository> [<refspec>…​]]`

### 准备工作
master分支取消跟踪远端分支
```
git checkout master
git branch --unset-upstream
```

### 1、不带参数
```
git pull

There is no tracking information for the current branch.
Please specify which branch you want to merge with.
See git-pull(1) for details.

    git pull <remote> <branch>

If you wish to set tracking information for this branch you can do so with:

    git branch --set-upstream-to=origin/<branch> master
```
说明不带参数时，git默认选择当前所在分支的跟踪分支；若没有设置跟踪分支，则放出上述提示

### 2、带远端仓库名
```
git pull origin

You asked to pull from the remote 'origin', but did not specify
a branch. Because this is not the default configured remote
for your current branch, you must specify a branch on the command line.
```
说明带远端仓库名时，若该远端仓库不是default remote的话，就会放出上述提示，没有设置跟踪分支，也属于这样的一种特殊情况

### 3、带远端仓库名及分支名
```
git pull origin dev
```
结果为将origin/dev merge到了当前所在分支即master中

### 结论1
- 设置远端分支，`git branch -u origin/master`，再进行一遍上面的实验，结果是相符的;
- `git pull`采用事先设置好的跟踪分支
- `git pull <远端仓库名>`，需要保证远端仓库名与设置的远端跟踪分支所属远端仓库一致
- `git pull <远端仓库名> <远端分支名>`则会无视跟踪关系进行merge，例如实验中，会将`origin/dev`merge到`master`上
- 指令格式中，仓库名与分支名以空格分隔

### 新增一个远端仓库
为方便起见，没有真的新增，url和origin指定的url是一样的，当前仍在master分支下
```
git remote add new <url>
git branch -u new/master
git branch -vv

 dev    169ed01 [origin/dev] 123
* master 2558521 [new/master: ahead 4] this is master two
```

### 4、在master分支下，不带参数
```
git pull -v

From github.com:MilesGO517/test
 = [up to date]      master     -> new/master
 = [up to date]      dev        -> new/dev
Already up to date.
```
可以看到，fetch的是new这个远端

### 5、在dev分支下，不带参数
```
git checkout dev
git pull -v

From github.com:MilesGO517/test
 = [up to date]      dev        -> origin/dev
 = [up to date]      master     -> origin/master
Already up to date.
```
可以看到，fetch的是origin这个远端

### 结论2
- git pull不带参数时，在内部执行git fetch <当前分支跟踪的远端仓库名>

### 设置origin/HEAD为origin/dev、设置master跟踪origin/master
```
git checkout master
vi .git/refs/remotes/origin/HEAD  # 按照要求修改
git branch -u origin/master
```

保证此时master与origin/dev在提交树中的不同分叉上，所以可以merge

### 6、在master分支下，带远端仓库名
```
git pull origin -v
From github.com:MilesGO517/test
 = [up to date]      master     -> origin/master
 = [up to date]      dev        -> origin/dev
Already up to date.
```

### 7、在master分支下，带远端仓库名及分支名
```
git pull origin dev -v
```
产生merge效果

### 结论3
- `git pull origin`, origin不会展开为origin/HEAD
- 若当前所在分支设置了远端跟踪分支
    - 若远端跟踪所属仓库是origin，则默认采用远端跟踪分支
    - 若远端跟踪所属仓库不是origin，则放出提示，需要具体指定分支名


## git-push
### 准备工作
```
git checkout dev
git branch --unset-upstream
git checkout master
git branch -u origin/dev
```
另外，master分支是origin/dev的后代

### 1、跟踪不同名远端分支，不带参数
```
git push

fatal: The upstream branch of your current branch does not match
the name of your current branch.  To push to the upstream branch
on the remote, use

    git push origin HEAD:dev

To push to the branch of the same name on the remote, use

    git push origin HEAD

To choose either option permanently, see push.default in 'git help config'.

git push origin HEAD:master  # 最终我们选择同步到同名远端分支
```

### 结论1
- git push不带参数，建议本地和跟踪的远端分支同名
- 如果跟踪的远端分支不同名，则会放出上述提示
- 如果一定要push到不同名的远端分支，则使用`git push <远端仓库名> <本地引用>:<远端分支名>`

### 2、不跟踪远端分支，不带参数
```
git checkout dev
git merge master
git push

fatal: The current branch dev has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin dev
```

### 结论2
- git push不带参数，要求一定要有跟踪的上游（远端）分支

### 3、不跟踪远端分支，带远端仓库与同名分支
```
git push origin dev

Total 0 (delta 0), reused 0 (delta 0)
To github.com:MilesGO517/test.git
   8618a06..ba38218  dev -> dev
```

### 4、不跟踪远端分支，带远端仓库与不同名分支
```
git checkout master
# 做出一些改动并提交
git checkout dev
git push origin master

Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 12 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 265 bytes | 265.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To github.com:MilesGO517/test.git
   ba38218..d13d660  master -> master
```

### 结论3
- git push带远端仓库与远端分支名，无视跟踪关系
- 将本地的同名分支push到远端

## git-fetch
https://git-scm.com/docs/git-fetch

```
git fetch [<options>] [<repository> [<refspec>…​]]
git fetch [<options>] <group>
git fetch --multiple [<options>] [(<repository> | <group>)…​]
git fetch --all [<options>]
```

git fetch比较简单，按照上面给出的指令格式使用即可，不再赘述


## git-merge
```
git merge [<options>] [<commit>…​]
```
注意这里的参数是commit，和前面的三个指令不同；  
`git merge origin`在执行前扩展为`git `