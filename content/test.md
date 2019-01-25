

## 测试驱动开发TDD

在某个 API 定义好之后，我们首先针对这个 API 写单元测试，这个时候测试是失败的，然后再完成 API 的代码来通过测试。这就是测试驱动开发的要点。

## 编写测试

在对一个Web服务进行测试时，基本思想就是先在测试环境下将这个web服务跑起来，之后再通过**httptest**包实现一系列函数，来对已绑定的API发送请求，判断是否可以达到预期效果。

一个web测试文件的基本结构如下(以gin为例):

```go
package test

import ...

// 全局应用对象，供各个函数使用
var g *gin.Engine

func init() {
    readconfig();
    dbinit();
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

