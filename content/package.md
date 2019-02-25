## 包管理

### Dep

Dep 包管理很简单，在项目目录下面跑 `dep init` 就可以了。如果是一个已经初始化好的项目要安装依赖，就跑一下 `dep ensure` 就可以。Dep 的安装等等可以看[官方文档](https://golang.github.io/dep/docs/introduction.html)。

我们在学 Go 的时候，一开始就会遇到 GOPATH 这个概念。Dep 还是 GOPATH 体系下面的，你的项目必须在 GOPATH 目录下面。Go 包的解析，首先会找当前目录下面的 vendor 文件夹，然后会找 GOPATH 下的包。所以 Dep 的原理就是解析出所有的包，把版本记录在一个文件里，然后把包都拉到项目下面的 vendor 文件夹下面。

### 关于翻墙
因为有些go的官方包是处在被墙状态的，直接`go get` 或者`dep init`等会出现阻塞或者timeout等情况。可以借鉴以下方案。
首先应该在本地起一个ssr客户端，并设置好相应的http代理端口，在这里我们设置为`127.0.0.1:8118`。
之后，在`.bashrc`或者`.zshrc`中alias如下命令：

```shell
alias proxy="
    export http_proxy=http://127.0.0.1:8118/;
    export https_proxy=http://127.0.0.1:8118/;
    export all_proxy=http://127.0.0.1:8118/;
    export no_proxy=localhost,127.0.0.0/8,::1;
    export HTTP_PROXY=http://127.0.0.1:8118/;
    export HTTPS_PROXY=http://127.0.0.1:8118/;
    export ALL_PROXY=http://10.0.0.117:8118/;
    export NO_PROXY=localhost,127.0.0.0/8,::1"
alias unproxy="
    unset http_proxy;
    unset https_proxy;
    unset all_proxy;
    unset no_proxy;
    unset HTTP_PROXY;
    unset HTTPS_PROXY;
    unset ALL_PROXY;
    unset NO_PROXY"
```
添加完成后 source ~/.bashrc 或 source ~/.bash_profile 或 source ~/.zshrc 或 重启终端。需要用 go mod、dep 前执行 proxy，不需要代理了执行 unproxy 或直接退出当前终端。

参考:
+ [Golang 项目被墙包的获取](https://g2ex.github.io/2018/05/25/go-dep-tips/)

### Go module

还是要介绍一下 Go module。Go module 很大的一个改变就是项目可以存在于 GOPATH 之外。

大家可以实际建一个文件夹试一下 Go module 的威力。

关于 Go Module 的一些介绍文章（更多文章可以直接 Google 搜 go module）：

+ [Introduction to Go Modules](https://roberto.selbach.ca/intro-to-go-modules/)
+ [Taking Go modules for a spin](https://dave.cheney.net/2018/07/14/taking-go-modules-for-a-spin)
+ [A gentle introduction to Golang Modules](https://ukiahsmith.com/blog/a-gentle-introduction-to-golang-modules/)
+ 一篇[中文笔记](http://zxc0328.github.io/diary/2019/01/2019-01-22.html)
