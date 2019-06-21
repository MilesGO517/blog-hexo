---
title: '【git】- 基本原理'
date: 2019-08-09 16:29:42
tags:
    - git
categories:
    - [git, 原理]
---
## 三个区域
- working directory
    - 也就是你当前所能操作的那些目录和文件
- history
    - 你所提交的所有记录，文件历史内容等等。git是个分布式版本管理系统，在你本地有项目的所有历史提交记录；文件历史记录；提交日志等等
- stage(index)
    - 暂存区域
    - 本质上是个文件，也就是.git/index

![](https://raw.githubusercontent.com/MilesGO517/images/master/2019/8/4.png)

## 文件的状态变化
![](https://raw.githubusercontent.com/MilesGO517/images/master/2019/8/5.png)

## 存储原理
### 概述
- git保存的不是文件的变化或者差异，而是一系列不同时刻的文件快照。
- 如果某一文件并没有变化，则该文件快照只需简单的使用一个指向前一版本的文件快照中该文件的指针即可。
- 怎么知道文件是否相同，或者说有没有变化？按照某种方式计算文件或者目录的校验和即可。
- 注意，文件名不影响文件的校验和。

<!--more-->

### 三种存储对象
包括blob对象、tree对象、commit对象

- blob对象
    - 用于表示一个文件
    - tree和blob对象可以看成是git内部采用的文件系统对象
    - 其组织结构和文件系统是很像的，很好理解
- tree对象
    - 用于表示一个目录
- commit对象
    - 用于表示一个提交
    - 每个提交对象都保存了顶层tree对象，对应工程根目录
    - commit对象包含若干个父提交对象
        - 首次提交产生的提交对象没有父对象
        - 普通提交操作产生的提交对象有一个父对象
        - 由多个分支合并产生的提交对象有多个父对象
    - commit对象之间会组织成DAG（有向无环图）的结构
    - commit对象包含的信息有
        - parent，父亲提交对象
        - tree, 根目录树对象
        - author，作者
        - committer，提交者
- 这些对象都保存在.git/objects/目录下，每一个对象都会生成1个40位的哈希值，前2位作为文件夹，后38位作为文件名
- 不同名，相同内容的文件，其哈希值相同，其blob对象也是共用的

- 以下图的例子理解这三种对象
![](https://raw.githubusercontent.com/MilesGO517/images/master/2019/8/git%E5%AD%98%E5%82%A8%E5%AF%B9%E8%B1%A1%E4%B8%BE%E4%BE%8B.png)

### 对应的物理存储
在.git/objects目录下，有很多目录
#### info目录
#### pack目录
#### 其它目录
均由两位字母或数字组成，内部是许多二进制文件，将两位目录名和文件名组合起来，即为40位存储对象校验和
这些文件保存的就是二进制的存储对象，直接使用cat指令将显示乱码
- 借助`git cat-file -t <SHA>`，可以查看该校验和指向的存储对象类型
- 借助`git cat-file -p <SHA>`，可以查看该校验和指向的存储对象保存的内容  

### 实验
#### 实验1
1、选择一个有过历史记录的git仓库，执行`git log`指令
![](https://raw.githubusercontent.com/MilesGO517/images/master/2019/8/1.png)

2、以当前master指向的提交记录为例
```
bash>> git cat-file -t 4c302fe27
commit   # 每次提交均用一个commit对象来表示

bash>> git cat-file -p 4c302fe27 
tree 05da74d9645c1b5405c87a4b97fe3cd2d0d40488  # 每个提交均包含一个tree对象，即根目录
parent 85be7ee6554233718a0efb522bb0151f623410a3
author MilesGO <guyijie@bytedance.com> 1565342317 +0800
committer MilesGO <guyijie@bytedance.com> 1565342317 +0800

This is commit information

bash>> git cat-file -p 05da74d
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391	1.txt
100644 blob 0cf6fd99b34631256d6fbebd880f7181c0221381	README
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391	a.txt

bash>> git cat-file -p 0cf6fd99b
This is a README file
```

#### 实验2
```
$ git init test
Initialized empty Git repository in C:/Users/MilesGO/Desktop/git/test/.git/
$ cd test
$ find .git/objects -type f
```
此时由于没有任何的提交记录, `find .git/objects -type f`指令的返回结果为空，这条指令会列出.git/objects目录下的所有文件  
接下来在根目录下创建一个README文件，文件内容hello git
```
$ echo 'hello git' >> README
find .git/objects -type f
```
返回结果仍然为空，说明在工作区域做出的改变，如果没有提交到暂存区或历史版本库，是不会产生任何的git对象的
```
git add README
$ find .git/objects -type f
.git/objects/8d/0e41234f24b6da002d962a26c2495ea16a425f
$ git cat-file -t 8d0e4123
blob
$ git cat-file -p 8d0e4123
hello git
```
`git cat-file -t`指令可以查看一个git对象的类型，`git cat-file -p`指令可以查看一个git对象的内容  
我们将README文件添加到暂存区之后，生成了一个blob文件，对应了暂存区中的README，其内容就是我们刚刚添加的`hello git`  
接下来我们将暂存区的README提交
```
$ git commit -m 'add README'
[master (root-commit) 29c0715] add README
 1 file changed, 1 insertion(+)
 create mode 100644 README

$ find .git/objects -type f
.git/objects/28/324fe5bad6e397de7a37aa7d8c94b6bf176b83
.git/objects/29/c07152991c7b3ee41f9cb5ac1eff1f26610665
.git/objects/8d/0e41234f24b6da002d962a26c2495ea16a425f

$ git cat-file -t 28324fe5
tree

$ git cat-file -t 29c07152
commit
```
可以看到新生成了1个commit对象和一个tree对象，分别查看这两个对象的内容
```
$ git cat-file -p 29c07152
tree 28324fe5bad6e397de7a37aa7d8c94b6bf176b83
author MilesGO <164173218@qq.com> 1560070361 +0800
committer MilesGO <164173218@qq.com> 1560070361 +0800

add README
```
commit对象表明了其对应的根目录tree为28324f，也就是刚才新生成的tree对象
```
$ git cat-file -p 28324f
100644 blob 8d0e41234f24b6da002d962a26c2495ea16a425f    README
```
这个tree对象中只包含了一个blob对象8d0e41，也就是一开始我们添加的README文件

#### 实验3
在实验1的基础上继续进行
在README文件中添加hello git again，commit后查看git对象情况
```
$ echo 'hello git again' >> README
$ git add README
$ git commit -m 'modify README'
[master 4a1d754] modify README
 1 file changed, 1 insertion(+)

$ find .git/objects -type f
.git/objects/28/324fe5bad6e397de7a37aa7d8c94b6bf176b83
.git/objects/29/c07152991c7b3ee41f9cb5ac1eff1f26610665
.git/objects/4a/1d75425a4cd5c3e206a4c69415c59802175039
.git/objects/8d/0e41234f24b6da002d962a26c2495ea16a425f
.git/objects/b9/641c1bd3e89e43cfaebf63305fc0a655635b50
.git/objects/f8/3422c24917dcb2c2f781c08d83ac2fbc363dd2

$ git cat-file -t 4a1d75
commit
$ git cat-file -t b9641c
tree
$ git cat-file -t f83422
blob
```
分别新增了1个commit，1个tree和1个blob对象
```
$ git cat-file -p 4a1d75
tree b9641c1bd3e89e43cfaebf63305fc0a655635b50
parent 29c07152991c7b3ee41f9cb5ac1eff1f26610665
author MilesGO <164173218@qq.com> 1560070808 +0800
committer MilesGO <164173218@qq.com> 1560070808 +0800

modify README
```
新的提交必然需要生成1个新的提交对象，其tree根目录对象为b9641c，也是新生成的，其parent对象为29c071，也就是第一次提交生成的提交对象  
```
$ 100644 blob f83422c24917dcb2c2f781c08d83ac2fbc363dd2    README
$ git cat-file -p b9641c
```
新生成的tree对象中，包含1条blob对象的信息
```
$ git cat-file -p f83422
hello git
hello git again
```
该blob对象也就是我们刚才修改后提交的README
到此为止，一共有2个commit对象，2个tree对象，2个blob对象

#### 实验4
创建src目录，在src目录下添加main.cpp，添加一些内容
```
$ mkdir src
$ echo 'hello world' >> src/main.cpp
$ git add --a
$ git commit -m 'add src/main.cpp'
[master 12c2c80] add src/main.cpp
 1 file changed, 1 insertion(+)
 create mode 100644 src/main.cpp

$ find .git/objects -type f
.git/objects/12/c2c80788dcd74a1cd739157c0a1011514d36b2
.git/objects/28/324fe5bad6e397de7a37aa7d8c94b6bf176b83
.git/objects/29/c07152991c7b3ee41f9cb5ac1eff1f26610665
.git/objects/3b/18e512dba79e4c8300dd08aeb37f8e728b8dad
.git/objects/3f/68ce3f72d2ee5bec285180c6fb0aa3a7eee4ff
.git/objects/4a/1d75425a4cd5c3e206a4c69415c59802175039
.git/objects/8d/0e41234f24b6da002d962a26c2495ea16a425f
.git/objects/b9/641c1bd3e89e43cfaebf63305fc0a655635b50
.git/objects/c5/fb7c2c039f5f48bf89f922267016e8702bead8
.git/objects/f8/3422c24917dcb2c2f781c08d83ac2fbc363dd2

```
新添加的对象有
```
12c2c8
3b18e5
3f68ce
c5fb7c
```
查看这些新添加的对象
```
$ git cat-file -t 12c2c8
commit
$ git cat-file -p 12c2c8
tree 3f68ce3f72d2ee5bec285180c6fb0aa3a7eee4ff
parent 4a1d75425a4cd5c3e206a4c69415c59802175039
author MilesGO <164173218@qq.com> 1560071508 +0800
committer MilesGO <164173218@qq.com> 1560071508 +0800

add src/main.cpp

$ git cat-file -t 3b18e5
blob
$ git cat-file -p 3b18e5
hello world

$ git cat-file -t 3f68ce
tree
$ git cat-file -p 3f68ce
100644 blob f83422c24917dcb2c2f781c08d83ac2fbc363dd2    README
040000 tree c5fb7c2c039f5f48bf89f922267016e8702bead8    src

$ git cat-file -t c5fb7c
tree
$ git cat-file -p c5fb7c
100644 blob 3b18e512dba79e4c8300dd08aeb37f8e728b8dad    main.cpp

```

#### 实验5
不同文件名，相同文件内容
```
$ echo 'hello world' >> main2.cpp
$ git add main2.cpp
$ git commit -m 'add main2.cpp'
[master 314b549] add main2.cpp
 1 file changed, 1 insertion(+)
 create mode 100644 main2.cpp
```
在这种情况下，会新增哪些git对象呢？
```
$ find .git/objects -type f
.git/objects/12/c2c80788dcd74a1cd739157c0a1011514d36b2
.git/objects/28/324fe5bad6e397de7a37aa7d8c94b6bf176b83
.git/objects/29/c07152991c7b3ee41f9cb5ac1eff1f26610665
.git/objects/31/4b5494eb06a9963f4e6f97075f2354c8a15f13
.git/objects/3b/18e512dba79e4c8300dd08aeb37f8e728b8dad
.git/objects/3e/c767e93a78c0ef5c4685eace73192bf6bc6848
.git/objects/3f/68ce3f72d2ee5bec285180c6fb0aa3a7eee4ff
.git/objects/4a/1d75425a4cd5c3e206a4c69415c59802175039
.git/objects/8d/0e41234f24b6da002d962a26c2495ea16a425f
.git/objects/b9/641c1bd3e89e43cfaebf63305fc0a655635b50
.git/objects/c5/fb7c2c039f5f48bf89f922267016e8702bead8
.git/objects/f8/3422c24917dcb2c2f781c08d83ac2fbc363dd2
```
其中314b54和3ec767是新增的
```
$ git cat-file -t 314b54
commit
$ git cat-file -p 314b54
tree 3ec767e93a78c0ef5c4685eace73192bf6bc6848
parent 12c2c80788dcd74a1cd739157c0a1011514d36b2
author MilesGO <164173218@qq.com> 1560072040 +0800
committer MilesGO <164173218@qq.com> 1560072040 +0800

add main2.cpp

$ git cat-file -t 3ec767
tree

$ git cat-file -p 3ec767
100644 blob f83422c24917dcb2c2f781c08d83ac2fbc363dd2    README
100644 blob 3b18e512dba79e4c8300dd08aeb37f8e728b8dad    main2.cpp
040000 tree c5fb7c2c039f5f48bf89f922267016e8702bead8    src

$ git cat-file -p c5fb7c
100644 blob 3b18e512dba79e4c8300dd08aeb37f8e728b8dad    main.cpp
```
main2.cpp和src/main.cpp共用了同一个blob对象

`git ls-files --stage`可以看到暂存区所有文件及其对应的blob文件
```
$ git ls-files --stage
100644 f83422c24917dcb2c2f781c08d83ac2fbc363dd2 0       README
100644 3b18e512dba79e4c8300dd08aeb37f8e728b8dad 0       main2.cpp
100644 3b18e512dba79e4c8300dd08aeb37f8e728b8dad 0       src/main.cpp
```
从这里也可以看到main2.cpp和src/main.cpp共用了同一个blob对象

## 引用原理
### 引用的本质
引用本质上是一个文件，文件中包含了指向某个提交对象的哈希值，意即该引用指向这个提交记录
- `引用`指的是对`提交记录`的引用  
- `提交记录`用`哈希值`唯一标识  
- 每个`引用`用一个文件表示，文件中保存`其引用的提交记录的哈希值`  

### 引用的分类
- 分支
    - 可变, 在不同的时刻可以指向`不同的提交记录`
    - 本地分支
        - 对应`.git/refs/heads/目录`中的文件
        - 每个`本地仓库`有多个`本地分支`
    - 远程分支
        - 对应`.git/refs/remotes/<远端仓库名>/目录`中的文件
        - 每个`本地仓库`可以对应多个`远端仓库`, 同时每个`远端仓库`可以有多个`远端分支`
- tag
    - 对应`.git/refs/tags/目录`中的文件
    - 不可变, 除非删除后重新创建, 否则总是指向`特定的提交记录`
    - 每个git仓库可以有多个`tag`
- HEAD
    - 对应`.git/HEAD`文件
    - 可变
        - 通常指向某个`本地分支`，即引用的引用
        - 也可以直接指向`某个提交记录`，称为`HEAD detached`, 即分离头指针状态
        - 也可以指向`tag`，git将这种情况也处理成`HEAD detached`
        - 也可以指向`远端分支`, git将这种情况也处理成`HEAD detached`
    - 每个git仓库只有一个`HEAD`
    - 表示当前`工作区`检出的文件(或者说文件在修改之前)是属于哪个`提交记录`的
    - `git checkout 指令`，就是在改变HEAD的指向
        - `git checkout 本地分支名`
        - `git checkout 提交记录哈希值`, detached
        - `git checkout 远端分支名`, detached
        - `git checkout tag名`, detached

### 分支的原理
- 分支本质上仅仅是指向提交对象的可变指针。 git的分支非常轻量，早建分支！多用分支！  
- git 的分支实质上仅是包含所指对象校验和（长度为 40 的 SHA-1 值字符串）的文件，所以它的创建和销毁都异常高效。 
- 创建一个新分支就相当于往一个文件中写入 41 个字节（40 个字符和 1 个换行符）  
- HEAD是当前指向的提交记录的指针，即HEAD指向了某一个提交记录，当前的工作区域中的内容就是该提交记录中的所有文件快照；  
- HEAD也可以指向某一个分支，表示当前在该分支下，前面提到了分支也是指向提交对象的指针，那此时HEAD就是指针的指针  
- 切换到某个分支下的本质是什么？即HEAD指向该分支名，该分支名又指向某个特定的提交记录  
- 一般情况下，我们只会在分支间来回切换，那HEAD通常指向的都是分支名；分离的HEAD指的就是，HEAD直接指向某个提交记录，而不指向分支名；

### 实验
```
$ git checkout master
Switched to branch 'master'

$ cat .git/HEAD
ref: refs/heads/master

$ cat .git/refs/heads/master
89d496d44f93d107a7eb404890cd15a14ba8845d
```

checkout master后, HEAD指向master, master指向89d496

```
$ git checkout milestone
Note: checking out 'milestone'.
You are in 'detached HEAD' state. 
HEAD is now at eecc5fe milestone

$ cat .git/refs/tags/milestone
eecc5fe060e5b86957f931fd931beae4f206d4eb

$ cat .git/HEAD
eecc5fe060e5b86957f931fd931beae4f206d4eb
``` 

checkout tag milestone后，HEAD指向eecc5f, detached HEAD

## 分支合并原理
两个提交记录A和B进行合并，它们的最近公共祖先为C，合并结果即为D，则在合并时（以下算法是我口嗨的，估计具体实现八九不离十）
- 对于某个文件x，A和B中只有一方包含
    - 则直接往D中加入x
- 对于某个文件x，A和B均包含
    - 若C中包含x
        - 若Ax=Cx，则D中加入Bx
        - 若Bx=Cx，则D中加入Ax
        - 若Ax和Bx，均与Cx不相等，则合并Ax和Bx，再加入D中
    - 若C中不包含x，则合并Ax和Bx，再加入D中

Ax和Bx在合并时，可能产生无法自动合并的冲突，需要人工解决冲突：使用编辑器打开冲突的文件，解决冲突后，保存文件，然后依次执行`git add`和`git commit`指令。

如果合并的两个提交记录，其中一个是另一个祖先节点，例如A是B的祖先节点，则只需将A移动到B的位置即可，git会显示
Fast-forward（快进合并）；此时最近公共祖先就是A本身。

### git merge
在分支A下，git merge 分支B，效果是
- 产生一个新的提交记录，该提交记录合并了A和B
- 分支A指向该新的提交记录
- 分支B指向不变
- 分支A原来指向的提交记录是当前分支A指向的提交记录的父节点
- 分支B指向的提交记录也是当前分支A指向的提交记录的父节点

### git rebase
Rebase 的优势就是可以创造更线性、整洁的提交历史。

在分支A下，git rebase 分支B，效果是
- 产生一个新的提交记录，该提交记录合并了A和B
- 分支A指向该新的提交记录
- 分支B指向不变
- 分支B指向的提交记录是当前分支A指向的提交记录的父节点

### merge和rebase的比较
git merge和git rebase的最终结果没有任何区别，只不过提交历史不同罢了，变基使得提交历史更加整洁。

一般我们这样做的目的是为了确保在向远程分支推送时能保持提交历史的整洁——例如向某个其他人维护的项目贡献代码时。 在这种情况下，你首先在自己的分支里进行开发，当开发完成时你需要先将你的代码变基到 origin/master 上，然后再向主项目提交修改。 这样的话，该项目的维护者就不再需要进行整合工作，只需要快进合并便可。

## git push原理
https://stackoverflow.com/questions/26005031/what-does-git-push-do-exactly

- git push 成功的条件
    - 远端分支可以按照fast-forward的方式更新到本地分支指向的提交，即远端分支指向的提交时本地分支指向的提交的祖先节点
    - 在不满足上述条件的情况下，可以使用-f参数进行强制更新
- git push 做了什么？
    - Copy the local commits（along with the related trees and blobs） that are not existed in the remote repo to the remote repo
    - Move the origin/master (both in your local git and in the remote git) to point to the same local/master commit
    - Push DOES NOT merge

