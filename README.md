# Go-Web-Application-Guideline

木犀团队 Go Web 工程规范

> 如何协作：具体每一节的规范在 content 文件夹里，选择具体的某一个文件，然后写在里面就行。最后在这个 README 里面写一个简介，和到具体规范文件的链接。


> 本规范深度参考了掘金小册-[基于 Go 语言构建企业级的 RESTful API 服务](https://juejin.im/book/5b0778756fb9a07aa632301e)

## 主要内容

### [配置规范](https://github.com/zxc0328/Go-Web-Application-Guideline/blob/master/content/config.md)

一个 Web 应用往往有一些配置信息，比如数据库信息，健康检查超时时间等等，这些信息可能来自配置文件，环境变量或者命令行参数，所以我们可以用一个单独的配置管理库来对配置进行管理。这里我们选用的是 [Viper](https://github.com/spf13/viper) 这个库。

### [错误处理规范](https://github.com/zxc0328/Go-Web-Application-Guideline/blob/master/content/error.md)

// TODO

我们的应用中往往会有错误和异常，这并不可怕，关键是我们要把这些信息合理的呈现和记录下来，以供我们分析和排查。合理的错误处理有两个好处：一，让 API 的消费者在发生错误时，可以接收到合适的 HTTP 状态码和应用内部的错误码。二，在发生错误时，把关键信息 Log 出来，可以让我们监控错误，分析错误，发现应用上线之后的问题。有了合适的错误处理，妈妈再也不担心我没法定位用户端的报错了。

### 测试规范

// TODO

在某个 API 定义好之后，我们首先针对这个 API 写单元测试，这个时候测试是失败的，然后再完成 API 的代码来通过测试。这就是测试驱动开发的要点。

### [包管理规范](https://github.com/zxc0328/Go-Web-Application-Guideline/blob/master/content/package.md)

使用官方的 Go Module 进行包管理。

### [数据库规范](https://github.com/zxc0328/Go-Web-Application-Guideline/blob/master/content/db.md)

后端应用中，很重要的一部分就是和数据库打交道。因此我们在如何使用数据库上，需要有一个统一的规范。


### [Log 规范](https://github.com/zxc0328/Go-Web-Application-Guideline/blob/master/content/log.md)

日志是后端应用运行时的日记本，是我们排查问题，监控问题的重要检索。因此，我们要在应用中做好日志的统一管理，方便日志平台进行日志分析。因为 Docker 容器的限制（Docker 默认会把日志按行分割），我们需要将一条日志压缩在一行，目前最方便的日志格式就是 JSON 了。我们把日志信息进行序列化，就变成了一个 JSON 字符串，完美契合 Docker 的日志规范。

### [API 文档规范](https://github.com/zxc0328/Go-Web-Application-Guideline/blob/master/content/api.md)

API 文档是前后端沟通接口的重要平台，但如何统一 API 文档的格式，写出高质量的 API 文档始终是一个问题。Swagger 的出现统一了 API 文档的标准。而自动生成 Swagger 文档的工具则进一步降低了写 API 文档的成本。


### Makefile 规范

// TODO

### 模板仓库

// TODO