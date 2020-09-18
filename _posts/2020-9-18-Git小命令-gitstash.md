---
layout:       post
title:        "git小命令-git stash"
date:         2020-9-18 11:50
catalog:      true
tags:
    - Git   
---

设想以下的场景：
前提条件：

某天你正在酣畅淋漓的写代码，看到IDE里花花绿绿的各种修改。git diff一看modified修改文件一大堆。你准备一顿敲打，最后来个大的commit（推荐尽量使用更小的commit记录）。


需求1:
但这个时候突然同事告诉你他推了最新的代码，里面他增加了一个公用的函数.你如果不想重复开发这个函数，那么你需要拉到这个代码。
但这个时候你有unstashed changes,是不能直接进行pull命令的。
你有两个选择：
1.心不甘情不愿的把这部分甚至编译都还不能通过的代码commit。再进行pull。
2.git stash将现在的这部分代码放入暂存区。再进行pull。

我选择后者的原因主要是一可以避免未通过编译或测试的代码弄脏commit记录，其次实在不知道如何填写这次commit的message。

那么简单贴一下git stash常用的几个命令吧。

## git stash
```bash
# top2 只需要掌握这两个命令，你就能覆盖80%的使用场景
git stash
# 从栈顶弹出最近压入暂存区的代码，不保留暂存区的副本
git stash pop

# 将stash的栈顶元素应用到代码，但保留暂存区的副本
git stash apply

# 展示stash栈内的所有元素。
git stash list

# 展示栈顶的代码和当前代码的区别
git stash show

# 从栈顶的那个提交直接创建新的分支，删除栈中的元素
git stash branch BRANCHNAME
```

以上的命令都默认对栈顶元素操作，如果想操作栈内的其他元素可以加参数。例如：`git stash show stash@{1}` 查看最近第二次压入栈的内容。
