---
title: Git troubleshooting
date: 2019-04-30 10:18:30
tags: 工具
---

## 别名

通常使用`git bash`提交代码时，需要`add`，`commit`，`push`三步，大多数情况下，`add`和`commint`可以合为一步，就可以使用`alias`这个配置，在`git config --help`中找到`alias`的说明：

> **`alias.* `**
>
> Command aliases for the git command wrapper - e.g. after defining "alias.last = cat-file commit HEAD", the invocation "git last" is equivalent to "git cat-file commit HEAD". To avoid confusion and troubles with script usage, aliases that hide existing Git commands are ignored. Arguments are split by spaces, the usual shell quoting and escaping is supported. A quote pair or a backslash can be used to quote them.

```sh
git config alias.ac '!git add -A & git commit -m'//定义ac命令
git ac 'feat: 新增ac命令'//调用ac命令
```

## 文件路径乱码

使用git时出现的文件路径乱码问题，翻阅帮助文档，找到如下配置：

> **`core.quotePath`**
>
> Commands that output paths (e.g. *ls-files*, *diff*), will quote "unusual" characters in the pathname by enclosing the pathname in double-quotes and escaping those characters with backslashes in the same way C escapes control characters (e.g. `\t` for TAB, `\n` for LF, `\\` for backslash) or bytes with values larger than 0x80 (e.g. octal `\302\265` for "micro" in UTF-8). If this variable is set to false, bytes higher than 0x80 are not considered "unusual" any more. Double-quotes, backslash and control characters are always escaped regardless of the setting of this variable. A simple space character is not considered "unusual". Many commands can output pathnames completely verbatim using the `-z`option. The default value is true.

可以看到，该属性对路径名做了处理，默认值为`true`，改为`false`并将`git bash`字符集改成`UTF-8`即可解决。