## 包管理

### Dep

Dep 包管理很简单，在项目目录下面跑 `dep init` 就可以了。如果是一个已经初始化好的项目要安装依赖，就跑一下 `dep ensure` 就可以。Dep 的安装等等可以看[官方文档](https://golang.github.io/dep/docs/introduction.html)。

我们在学 Go 的时候，一开始就会遇到 GOPATH 这个概念。Dep 还是 GOPATH 体系下面的，你的项目必须在 GOPATH 目录下面。Go 包的解析，首先会找当前目录下面的 vendor 文件夹，然后会找 GOPATH 下的包。所以 Dep 的原理就是解析出所有的包，把版本记录在一个文件里，然后把包都拉到项目下面的 vendor 文件夹下面。

### Go module

还是要介绍一下 Go module。Go module 很大的一个改变就是项目可以存在于 GOPATH 之外。

大家可以实际建一个文件夹试一下 Go module 的威力。

关于 Go Module 的一些介绍文章（更多文章可以直接 Google 搜 go module）：

+ [Introduction to Go Modules](https://roberto.selbach.ca/intro-to-go-modules/)
+ [Taking Go modules for a spin](https://dave.cheney.net/2018/07/14/taking-go-modules-for-a-spin)
+ [A gentle introduction to Golang Modules](https://ukiahsmith.com/blog/a-gentle-introduction-to-golang-modules/)
+ 一篇[中文笔记](http://zxc0328.github.io/diary/2019/01/2019-01-22.html)