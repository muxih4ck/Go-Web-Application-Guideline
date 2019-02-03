

## 测试驱动开发TDD

在某个 API 定义好之后，我们首先针对这个 API 写单元测试，这个时候测试是失败的，然后再完成 API 的代码来通过测试。这就是测试驱动开发的要点。

在一个基于 Gin 的 Go HTTP 服务中，我们需要对 handler，model，service 以及 util/helper 函数进行测试。总的来说，测试是针对函数的。针对普通的函数，我们可以直接调用，看返回值。如果是 handler 这种函数，我们可以把 handler 挂到一个 gin 路由上，然后用 httptest 这个包，发送一个请求，然后看返回值。

关于 Go 测试的基础知识可以看[这里](https://static.muxixyz.com/test.pdf)。

### 测试初始化逻辑

在开始写测试之前，我们需要进行一些**公共**的初始化逻辑，比如初始化数据库和配置等等。这个时候我们可以在顶级目录写一个 `common_test.go`，然后在里面写一个 `TestMain`。

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
	//Set Gin to Test Mode
	gin.SetMode(gin.TestMode)
	fmt.Println("fewfewf")
	// Run the other tests
	os.Exit(m.Run())
}

```

如果 Go 发现有一个 TestMain 函数，这个函数会在所有测试执行之前执行。所以我们可以在这里做一些初始化工作。在具体每个函数的测试里我们只需要关注具体的测试逻辑就行了。

> 关于测试的初始化可以参考这个 [Stack Overflow 回答](https://stackoverflow.com/questions/23729790/how-can-i-do-test-setup-using-the-testing-package-in-go)。

### 编写 Handler 测试

```go 
// Test that a GET request to the login page returns
// an HTTP error with code 401 for an authenticated user
func TestSomeHandler(t *testing.T) {
	// Create a response recorder
	w := httptest.NewRecorder()

	// Get a new router
	r := getRouter(true) // util，见下文代码

	// Set the token cookie to simulate an authenticated user
	http.SetCookie(w, &http.Cookie{Name: "token", Value: "123"})

	// Define the route similar to its definition in the routes file
	r.GET("/u/login", ensureNotLoggedIn(), showLoginPage)

	// Create a request to send to the above route
	req, _ := http.NewRequest("GET", "/u/login", nil)
	req.Header = http.Header{"Cookie": w.HeaderMap["Set-Cookie"]}

	// Create the service and process the above request.
	r.ServeHTTP(w, req)  // 用这个 API 来进行请求，不用 r.Run(8000) 之类的

	// Test that the http status code is 401  状态码 或者 拿到 json string 反序列化到 结构体。看具体需求，一般如果是测试异常情况，直接看状态码就差不多。正常情况就那结构体来反序列化一下看看是否正确
	if w.Code != http.StatusUnauthorized {
		t.Fail()
	}
}
```

getRouter

```go
func getRouter(withTemplates bool) *gin.Engine {
	r := gin.Default()
	if withTemplates {
		r.LoadHTMLGlob("templates/*")
		r.Use(setUserStatus())
	}
	return r
}
```
### 编写普通函数测试

按小册里来的。Model 测试需要有个 mock 的结构体


### 编写测试

在对一个Web服务进行测试时，基本思想就是先在测试环境下将这个web服务跑起来，之后再通过**httptest**包实现一系列函数，来对已绑定的API发送请求，判断是否可以达到预期效果。

一个web测试文件的基本结构如下(以gin为例):

```go
package test

import ...

// 全局应用对象，供各个函数使用
var g *gin.Engine

func init() {
    ReadConfig();
    DbInit();
    g = gin.New()
    BindMiddlewares();
}


func TestAPIOne(t *testing.T) {
    // 不包含host的URI
    uri := "/v1/user"
    
    // 构建JSON等需要的部分 
    u := user.CreateRequest{
        Username:  "username",
        Password:  "password",
    }
    jsonByte, err := json.Marshal(u)
    if err != nil {
        t.Errorf("Test Error: %s", err.Error())
    }
    // 使用httptest包构建对web服务的请求 req
    req := httptest.NewRequest(http.MethodPost, uri, bytes.NewReader(jsonByte))
    // w 用来记录测试的response
    w := httptest.NewRecorder()
    // 使我们的web服务处理该http请求
    g.ServeHTTP(w, req)
    // 读取结果，根据结果来判断成功与否
    result := w.Result()
    // 读取body
    defer result.Body.Close()
    body,_ := ioutil.ReadAll(result.Body)
    if result.StatusCode != http.StatusOK || string(body) != "ok!" {
        t.Errorf("Test Error: StatusCode Error:%d", result.StatusCode)
    }
}

TestAPITwo(t *testing.T) {
    
}

TestAPIThree(t *testing.T) {
    
}
...
```

