---
title: 如何用GVM管理Go项目
date: 2020-01-13 10:35:34
tags:
---
>使用 Go 版本管理器管理多个版本的 Go 语言环境及其模块

![如何用 GVM 管理 Go 项目](https://cdn.learnku.com/uploads/images/201910/18/36517/ZDHqtzEHMw.png!large)

Go 语言版本管理器（[GVM](https://github.com/moovweb/gvm)）是管理 Go 语言环境的开源工具。GVM 「pkgsets」 支持安装多个版本的 Go 并管理每个项目的模块。它最初由 [Josh Bussdieker](https://github.com/jbussdieker) 开发，GVM（像它的对手 Ruby RVM 一样）允许你为每个项目或一组项目创建一个开发环境，分离不同的 Go 版本和包依赖关系，来提供更大的灵活性，以防不同版本造成的问题。

有几种管理 Go 包的方式，包括 Go 1.11 内置于 Go 中的 Modules。我发现GVM简单而直观，即使我不使用它来管理包，我仍然会使用它来管理不同的 Go 版本。
<!--more-->
## 安装 GVM
安装 GVM 很简单。[GVM库](https://github.com/moovweb/gvm#installation)安装文档指导您下载安装脚本并将其导入 Bash。

`bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)`

尽管越来越多的人采用这种安装方法，但最好还是在安装之前先看看安装程序在做什么。以 GVM 为例，安装脚本执行过程：
1.  检查相关依赖
2.  克隆 GVM 库
3.  使用 shell 脚本:
    -   安装 Go 语言
    -   管理 GOPATH 环境变量
    -   在 bashrc 、zshrc 或配置文件中添加一行内容
    
	
如果您想再次检查它在做什么，您可以克隆库并检查 shel l脚本，然后运行 `./binscripts/gvm-installer` 使用本地脚本进行设置。

*注意:* 由于 GVM 可用于下载和编译新的 Go 版本，因此会有一些预期的依赖项，如 Make，Git 和 Curl。你可以在[GVM 的 README](https://github.com/moovweb/gvm/blob/master/README.md) 中找到完整的发行版列表。

## 使用 GVM 安装和管理 Go 版本

一旦安装了 GVM，您就可以开始使用它来安装和管理不同版本的 Go。`gvm listall` 命令显示了可以下载和编译的可用的 Go 版本：
```
[chris@marvin ]$ gvm listall
$ gvm listall

gvm gos (available)

   go1
   go1.0.1
   go1.0.2
   go1.0.3

<输出截断>
```
安装特定的 Go 版本就想 `gvm install <version>` 一样简单，其中 `<version>` 是 `gvm install` 命令返回的版本之一。

假设您正在处理一个使用 Go 1.12.8 版本的项目。你可以使用 `gvm install go1.12.8` 命令来安装这个版本:
```
[chris@marvin]$ gvm install go1.12.8
Installing go1.12.8...
 * Compiling...
go1.12.8 successfully installed!
```
输入 `gvm list`，你会看到 Go 1.12.8 版本与系统 Go 版本（使用操作系统的软件包管理器打包的版本）同时存在：
```
[chris@marvin]$ gvm list
gvm gos (installed)
   go1.12.8
=> system
```
GVM 仍然再使用系统的 Go 版本，通过它旁边的 **=>** 符号来表示。你可以使用 `gvm use` 命令来切换到新安装的 go1.12.8版本：
```
[chris@marvin]$ gvm use go1.12.8
Now using version go1.12.8
[chris@marvin]$ go version
go version go1.12.8 linux/amd64
```
GVM使管理已安装的Go版本变得极其简单，但它不止如此！

## 使用 GVM pkgset

在开箱即用的情况下，Go 以一种出色而又令人沮丧的方式管理包和模块。默认情况下，如果你 `go get` 获取一个包，它会被下载到  `$GOPATH` 目录中的 `src` 和 `pkg` 目录下；然后你可以使用 `import` 将其引入到你的 Go 程序中。这使得获取包变得很容易，特别是对于没有特权的用户，不需要 `sudo` 或 root 特权(很像 Python 中的`pip install --user`)。然而，在不同的项目中管理相同包的不同版本是困难的。

有许多方法可以尝试修复或缓解这个问题，包括实验性的 Go Modules (在 Go v1.11 版本中增加了初步支持)和 [Go dep](https://golang.github.io/dep/)(一个「官方实验」，被正在被 Go Modules 所替代)。在我发现GVM之前，为了确保隔离，我会在自己的Docker容器中构建和测试Go项目。

GVM 通过使用「pkgsets」将项目的新目录附加到 Go 安装版本的默认 `$GOPATH`，很像在 Unix/Linux 系统上工作的`$PATH`，很好地完成了项目之间包的管理和隔离。

很容易想象这是如何运行的。首先，安装新版本的Go 1.12.9：
```
[chris@marvin]$ echo $GOPATH
/home/chris/.gvm/pkgsets/go1.12.8/global
[chris@marvin]$ gvm install go1.12.9
Installing go1.12.9...
 * Compiling...
go1.12.9 successfully installed
[chris@marvin]$ gvm use go1.12.9
Now using version go1.12.9
```
当 GVM 被告知使用一个新版本时，它将会更换一个新的 `$GOPATH`，**gloabl** pkgset 将默认使用该版本：
```
[chris@marvin]$ echo $GOPATH
/home/chris/.gvm/pkgsets/go1.12.9/global
[chris@marvin]$ gvm pkgset list
gvm go package sets (go1.12.9)
=>  global
```
尽管默认情况下没有安装额外的包，但是 global pkgset 中的包对于使用这个特定版本 Go 的任何项目都是可用的。

现在，假设您正在启动一个新项目，它需要一个特定的包。首先，使用 GVM 创建一个名为 `introToGvm` 的新的pkgset：
```
[chris@marvin]$ gvm pkgset create introToGvm
[chris@marvin]$ gvm pkgset use introToGvm
Now using version go1.12.9@introToGvm
[chris@marvin]$ gvm pkgset list
gvm go package sets (go1.12.9)
    global
=>  introToGvm
```
如上所述，pkgset 的一个新目录被添加到 `$GOPATH`：
```
[chris@marvin]$ echo $GOPATH
/home/chris/.gvm/pkgsets/go1.12.9/introToGvm:/home/chris/.gvm/pkgsets/go1.12.9/global
```
将目录更改为预先设置的 `introToGvm` 路径，并检查目录结构，然后利用这个机会体验一下 `awk` 和 `bash` 的乐趣:
```
[chris@marvin]$ cd $( awk -F':' '{print $1}' <<< $GOPATH )
[chris@marvin]$ pwd
/home/chris/.gvm/pkgsets/go1.12.9/introToGvm
[chris@marvin]$ ls
overlay  pkg  src
```
注意，新目录看起来很像普通的 `$GOPATH`。新的 Go 包可以使用与 Go 相同的 `Go get` 下载命令，且被添加到 pkgset中。

例如，使用以下代码获得 `gorilla/mux` 包，然后检查 pkgset：
```
[chris@marvin]$ go get github.com/gorilla/mux
[chris@marvin]$ tree
[chris@marvin introToGvm ]$ tree
.
├── overlay
│   ├── bin
│   └── lib
│       └── pkgconfig
├── pkg
│   └── linux_amd64
│       └── github.com
│           └── gorilla
│               └── mux.a
src/
└── github.com
    └── gorilla
        └── mux
            ├── AUTHORS
            ├── bench_test.go
            ├── context.go
            ├── context_test.go
            ├── doc.go
            ├── example_authentication_middleware_test.go
            ├── example_cors_method_middleware_test.go
            ├── example_route_test.go
            ├── go.mod
            ├── LICENSE
            ├── middleware.go
            ├── middleware_test.go
            ├── mux.go
            ├── mux_test.go
            ├── old_test.go
            ├── README.md
            ├── regexp.go
            ├── route.go
            └── test_helpers.go
```
如您所见，`gorilla/mux` 按照预期添加到了 pkgset  的 `$GOPATH` 目录，现在它可以与使用该 pkgset 的项目一起使用了。

## GVM 让 Go 管理变得轻而易举

GVM 是以一种直观的、非侵入性的方式来管理 Go 版本和包的。它可以单独使用，也可以使用 GVM 的 Go 版本管理功能与其他 Go 模块管理技术结合使用。通过 Go 版本和包依赖关系隔离项目使开发更容易，并减少了管理版本冲突的复杂性，而GVM使这变得轻而易举。
>本文译自 [opensource](https://opensource.com/article/19/10/go-introduction-gvm)
