## Logging 规范

我们使用的 Log 库是[lexkong/log](https://github.com/lexkong/log)，关于这个库的具体使用介绍可以看[这里](https://static.muxixyz.com/logging.pdf)。

在 Logging 规范这节我们主要想说的主要是，什么时候需要打 Log？应该打什么级别的 Log？Log 的内容是什么？下面我们一个个来看。

### 什么时候需要打 Log？

很简单，唯一一个必须要打 Log 的地方，就是发生错误的地方。比如查数据库出了问题：

```
if err != nil {
    // 打日志
    log.Error(err.Error(), lager.Data{"X-Request-Id": util.GetReqID(c), "Code": errno.ErrEntityNotFound.Code})
	SendInternalError(c, errno.ErrEntityNotFound, nil)
	return
}
```

> 在写代码的时候，一定要考虑到 error 的情况。查数据库这种出错，爬虫爬数据也会出错，我们需要在代码里对这些 error 都进行处理。Error 不可怕，不知道发生了哪些 Error 才可怕。


### Log 的内容和级别？

日志级别有这几种：

`DEBUG, INFO, WARN, ERROR, FATAL`

DEBUG 就是开发时用的调试日志。INFO 是信息，比如一个请求过来了，就打印一个 log.Info。WARN 是警告，FATAL 和 ERROR 都是错误，区别是 ERROR 是可以容忍的错误，会在 HTTP Response 里返回过去。而 FATAL 是致命错误，基本上意味着应用马上就要退出了（挂了）。

以上文中的日志为例，对于一个 Error 级别的 log 我们打 log 的内容主要包括错误描述，请求 id，内部错误码。请求的 id 可以返回给请求的调用者，这样外面就可以根据请求 id 来方便的查找相关错误日志了。

> Go 的内置 error 类型只有一个 Error 方法用来返回一个描述错误的字符串。如果需要 stacktrace，需要[自行对错误做封装](https://blog.bugsnag.com/go-errors/)。目前来看，在必要的地方（一般是 I/O）做好错误处理，就足以定位问题了。

