---
title: Git troubleshooting
date: 2019-04-30 10:18:30
tags: 工具
---

# 别名

通常使用`git bash`提交代码时，需要`add`，`commit`，`push`三步，大多数情况下，`add`和`commint`可以合为一步，就可以使用`alias`这个配置，在`git config --help`中找到`alias`的说明：

> **alias.* **
>
> Command aliases for the git command wrapper - e.g. after defining "alias.last = cat-file commit HEAD", the invocation "git last" is equivalent to "git cat-file commit HEAD". To avoid confusion and troubles with script usage, aliases that hide existing Git commands are ignored. Arguments are split by spaces, the usual shell quoting and escaping is supported. A quote pair or a backslash can be used to quote them.

```sh
git config alias.ac "!git add -A && git commit -m"//定义ac命令
git ac 'feat: 新增ac命令'//调用ac命令
```

