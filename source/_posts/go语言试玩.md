---
title: go语言试玩
date: 2019-11-26 18:27:12
tags: go
---

# go tour

[go语言官方文档]( https://golang.google.cn/doc/ )中推荐了一个入门教程：A Tour of Go，根据官方描述，很简单就能安装成功，但是由于众所周知的原因，单单是下载就不容易。终于下载完成准备运行，报出如下错误：

```
Couldn't find tour files: could not find go-tour content; check $GOROOT and $GOPATH
```

错误原因：

```properties
GO111MODULE = on
```

由于启用了go module设置，tour的源码被下载到` $GOPATH/pkg/mod/golang.org/x/tour@v0.0.0-20191002171047-6bb846ce41cd `，但是当运行tour启动命令时，程序会到`src`目录下查找所需的文件，就出现了找不到文件的错误信息。

解决方案1：

将源码文件夹拷贝到`$GOROOT`或`$GOPATH`的`src`目录下，再运行`tour`

解决方案2：

修改源码路径下的`local.go`文件

```go
func findRoot() (string, error) {
    ...

    if os.Getenv("GO111MODULE") == "on" {
        pkgDir := filepath.Join(os.Getenv("GOPATH"), "pkg", "mod")
        cfg := &packages.Config{Mode: packages.LoadFiles, Dir: pkgDir}
        pkgs, _ := packages.Load(cfg, basePkg)
        for _, pkg := range pkgs {
            for _, goFile := range pkg.GoFiles {
                root := filepath.Dir(goFile)
                if isRoot(root) {
                    return root, nil
                }
            }
        }
    }
    return "", fmt.Errorf("could not find go-tour content; check $GOROOT and $GOPATH")
}
```

[参考文献]( https://stackoverflow.com/questions/57686808/error-about-running-go-tour-localy-couldnt-find-tour-files-could-not-find-go )