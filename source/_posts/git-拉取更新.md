---
title: '【git】- 拉取更新'
date: 2019-08-09 15:53:29
tags:
    - git
categories:
    - [git, 最佳实践]
---

## 常用
- master开发模式下，和中心仓储同步时，不要使用`git pull`
    - 请使用 `git pull --rebase`
    - 或 `git fetch; git rebase origin/master`以保持线性历史. 
- 在其它分支开发时，最好也如此

<!--more-->

## 实验
### git pull
一个远端仓库，分别对应两个本地仓库A和B，均在master分支上改动main.cpp
main.cpp原本内容
```
#include <iostream>
int main() {
    std::out << "hello world" << std::endl;
    return 0;
}
```

A改动如下，并先push到了远端仓库
```
#include <iostream>
int main() {
    std::out << "hello world" << std::endl;
    std::out << "hello main" << std::endl;
    return 0;
}
```

B改动如下
```
#include <iostream>
int main() {
    std::out << "hello world" << std::endl;
    std::out << "hello backup" << std::endl;
    return 0;
}
```

B尝试push但是失败
```
bash<< git push origin master
To github.com:MilesGO517/test.git
 ! [rejected]        master -> master (fetch first)
error: 推送一些引用到 'git@github.com:MilesGO517/test.git' 失败
提示：更新被拒绝，因为远程仓库包含您本地尚不存在的提交。这通常是因为另外
提示：一个仓库已向该引用进行了推送。再次推送前，您可能需要先整合远程变更
提示：（如 'git pull ...'）。
提示：详见 'git push --help' 中的 'Note about fast-forwards' 小节。
```

原因在于，origin/master ahead 1 本地master
```
bash<< git branch -vv
* master 11b2db8 [origin/master: 领先 1] add hello backup
```

需要先拉取远端的更新，很多人可能就直接`git pull`了`(git pull = git fetch + git merge)`，会产生如下的效果
```
bash<< pull
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (1/1), done.
展开对象中: 100% (3/3), 完成.
remote: Total 3 (delta 1), reused 3 (delta 1), pack-reused 0
来自 github.com:MilesGO517/test
   2b30fcd..5bc8dc5  master     -> origin/master
自动合并 main.cpp
冲突（内容）：合并冲突于 main.cpp
自动合并失败，修正冲突然后提交修正的结果。 
```

进入main.cpp，合并冲突，结果如下
```
#include <iostream>

int main() {
        std::out << "hello world" << std::endl;
        std::out << "heelo backup" << std::endl;
        std::out << "hello main" << std::endl;
        return 0;
}
```

然后提交merge结果并提交到远端仓库
```
bash<< git commit -am 'merged'
bash<< git push origin master
```

用可视化工具查看历史记录
![](https://raw.githubusercontent.com/MilesGO517/images/master/2019/8/3.png)

`git pull = git fetch + git merge`时，`git merge`这一步使得提交历史不干净；
比较推荐的做法是`git fetch、git rebase origin/master`，或者`git pull --rebase`，实验步骤与上述相同，最后可以观察到线性的提交历史，分叉的提交记录相当于被抛弃了

