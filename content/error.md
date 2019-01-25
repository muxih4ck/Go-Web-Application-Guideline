## 错误处理规范


### 常见的错误类型以及状态码规范

**401：Unauthorized**

用户提供的认证信息是错误的，比如用户名或者密码错误。

**403： Forbidden**

用户已经登录，但是没有访问某个资源的权限。通常用在用户无权限的情况下。

**400：Bad Request**

请求的参数（query string 或者 body）格式不正确，通常在 Go 中，就是请求 handler 里 bind 请求参数失败时返回的状态码。

**500：Internal Server Error**

服务端内部错误，如果不是上面几个错误，500 这个错误码可以被用在所有的服务端错误中，比如数据库查询的报错，网络请求错误等等。


我们在代码中可以针对以上几种情况，封装公共的 SendResponse 函数：

```
func SendBadRequest(c *gin.Context, err error, data interface{}, cause string) {
	code, message := errno.DecodeErr(err)
	log.Error(message lager.Data{"X-Request-Id": util.GetReqID(c), "Code": code, "cause": cause})
	if err != nil {
		c.JSON(400, Response{
			Code:    code,
			Message: message,
			Data:    data,
		})
	}
}

func SendInternalError(c *gin.Context, err error, data interface{}, cause string) {
	 code, message := errno.DecodeErr(err)
    log.Error(message, lager.Data{"X-Request-Id": util.GetReqID(c), "Code": code, "cause": cause})
	if err != nil {
		c.JSON(500, Response{
			Code:    code,
			Message: message,
			Data:    data,
		})
	}
}
```

在业务代码里只要调这些公共函数就可以了。

### 通用的返回 JSON 格式

光是 HTTP 状态码本身，无法提供所有的信息。如果是 500 错误，我们还需要知道具体的错误是什么。所以我们需要规定一个返回 JSON 格式规范。

```
{
  code: 0,
  message: "success",
  data: {}
}
```

code 代表一个业务的错误代码，0 代表 ok，其他的代码代表有错误或者异常。message 是一个描述信息。data 就是返回的数据。我们可以定义一些错误代码常量，比如这样：

```
	ErrValidation     = &Errno{Code: 20001, Message: "Validation failed."}
	ErrDatabase       = &Errno{Code: 20002, Message: "Database error."}
	ErrToken          = &Errno{Code: 20003, Message: "Error occurred while signing the JSON web token."}
	ErrEntityNotFound = &Errno{Code: 20004, Message: "Entity not found."}
	ErrAccessDenied   = &Errno{Code: 20005, Message: "Not allowed."}
```

这些都是具体的错误点，一个 500 不足以解释问题，所以我们用一套自定义的状态码来对错误进行进一步的解释。

### 业务逻辑异常 vs 服务端错误

我们要注意区分业务逻辑异常和服务端错误。

业务逻辑异常指业务中存在的分支情况，是可预测的，完全正常的。比如登录时，如果用户还没有绑定手机，这个时候可以返回 200，然后在返回的 JSON 的 code 字段中指定一个业务异常代码。在代码中可以根据 code 字段来判断这次登录的结果。


服务端错误是不正常的，对于服务端错误，我们返回的 HTTP 状态码一定是 500。这一方面是遵守 HTTP 的语义。一方面是为了把服务端错误和其他的情况区分开来。在调用方我们可以针对 500 状态码的返回做统一处理。

这两种情况的共同点是都会返回一个自定义的 code，但 HTTP 状态码 200 VS 500 的区别已经说明了这两种情况有着本质的不同。
