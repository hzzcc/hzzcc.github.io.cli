---
title: git alias 设置
date: 2016-05-29 09:11:11
tags: [git, "关联命令"]
toc: false

---
1. 在用户目录下创建文件

	```
    touch ~/.gitconfig
    ```

2. 写入如下内容

	```
    [alias]
        st = status
        ci = commit
        co = checkout
        br = branch
        unstage = reset HEAD --
        last = log -1 HEAD
    ```
