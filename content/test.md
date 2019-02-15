## 测试规范

### 基础

在某个 API 定义好之后，我们首先针对这个 API 写单元测试，这个时候测试是失败的，然后再完成 API 的代码来通过测试。这就是测试驱动开发的要点。

在一个基于 Gin 的 Go HTTP 服务中，我们需要对 handler，model，service 以及 util/helper 函数进行测试。总的来说，测试是针对函数的。针对普通的函数，我们可以直接调用，看返回值。如果是 handler 这种函数，我们可以把 handler 挂到一个 gin 路由上，然后用 httptest 这个包，发送一个请求，然后看返回值。

关于 Go 测试的基础知识可以看[这里](https://static.muxixyz.com/test.pdf)。

### 测试初始化逻辑

首先要理清一个点，Go 的测试是针对某一个 package 的。不同 package 之间的测试是并行执行的，只有同一个文件里面的各个测试函数才是按顺序执行的。

在开始写测试之前，我们需要进行一些**公共**的初始化逻辑，比如初始化数据库和配置等等。这个时候我们可以在测试中写一个 `TestMain`。

```go
var (
	cfg = pflag.StringP("config", "c", "", "apiserver config file path.")
)

// This function is used to do setup before executing the test functions
func TestMain(m *testing.M) {

	pflag.Parse()

	// init config
	if err := config.Init(*cfg); err != nil {
		panic(err)
	}

	// init db
	model.DB.Init()
	defer model.DB.Close()

	// Run the other tests
	os.Exit(m.Run())
}

```

如果 Go 发现有一个 TestMain 函数，这个函数会在所有测试执行之前执行。所以我们可以在这里做一些初始化工作。在具体每个函数的测试里我们只需要关注具体的测试逻辑就行了。

> 再次强调，测试是以 package 为单位来执行的，一个 package 可以有多个测试文件，这些文件的执行顺序是没有保证的。所以 `TestMain` 里面要做的就是针对同一个 package 下面所有测试的初始化工作。

> 关于测试的初始化可以参考这个 [Stack Overflow 回答](https://stackoverflow.com/questions/23729790/how-can-i-do-test-setup-using-the-testing-package-in-go)。

### 编写 Handler 测试

下面我们就来讲讲如何编写 API 的测试。API 测试属于 End to End （端到端）测试，因为我们会模拟一个客户端来请求这个服务端的 API，然后看结果是不是正确。

下面以 `/login` API 为例：

```go
var (
	g           *gin.Engine
	tokenString string
)

func TestLogin(t *testing.T) {
	// 获取一个新的 gin 路由实例
	g := getRouter(true)

	uri := "/login"

	// 构造http request body
	u := CreateRequest{
		Username: "admin",
		Password: "admin",
	}
	jsonByte, err := json.Marshal(u)
	if err != nil {
		t.Errorf("Test Error: %s", err.Error())
	}

	// util 封装的一个发请求的 helper，代码在 /util/test.go
	w := util.PerformRequestWithBody(http.MethodPost, g, uri, jsonByte, "")

	var data LoginResponse

	// 拿到请求，看这个请求是否可以被正确解析（拿返回 JSON 的结构体去 Unmarshal）
	if err := json.Unmarshal([]byte(w.Body.String()), &data); err != nil {
		t.Errorf("Test error: Get LoginResponse Error:%s", err.Error())
	}

	// token 拿到后保存到变量里供后面使用
	tokenString = data.Data.Token

    // 看请求的状态码是否是 200
	if w.Code != http.StatusOK {
		t.Errorf("Test Error: StatusCode Error:%d", w.Code)
	}
}
```

其他 API 的测试就不一一介绍了。具体可以看 `/handler/user/user_test.go`。我们从中可以收获到一些点：

+ 几个相关的 API 可以写在同一个文件里测试，这样就可以用前一个测试的返回来作为下一个测试的输入（比如登录之后可以拿到 token）
+ 测试错误处理也是测试的一部分，比如可以测试在用户名密码错误的情况下，返回是否正确。如果只测试正确的情况，那测试覆盖率其实是比较低的
+ 有些鸡生蛋蛋生鸡的情况下（比如登录 API 需要账户，但账户创建需要登录之后的 token），我们会提前在数据库里插入一些测试数据，比如我们的模板项目里，数据库中是已经有一个用户了的。同理，如果一个包测试需要依赖另外包里的 API，我们也会选择提前插入 mock 数据。单个测试测的主要还是 某个 API 本身的逻辑，我们假设其他的 API 都是正常的。

### 编写普通函数测试

普通函数（纯函数）其实就调用一下，然后看输出就可以了。按[小册](https://static.muxixyz.com/test.pdf)里的来。Model 测试需要有个 mock 的结构体

### CI 持续集成