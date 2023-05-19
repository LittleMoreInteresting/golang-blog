+++
title = "go-zero 单体应用实践（二）"
description = "单体应用实践"
weight = 2
url = "/go-zero/02.html"
+++
# 中间件使用
> 在go-zero中，中间件可以分为路由中间件和全局中间件，路由中间件是指某一些特定路由需要实现中间件逻辑，其和jwt类似，没有放在jwt:xxx下的路由不会使用中间件功能， 而全局中间件的服务范围则是整个服务。

## 路由中间件
1、编辑 api 文件 userlogin/userlogin.api  生命接口需要添加的中间件，多个中间件用逗号分隔
```shell
@server(
	middleware : Tagging,Version
)
service userlogin-api {
	@handler Tags
	get /api/tags returns (TagResponse)
}
```

2、goctl api 命令重新执行 生成middleware 文件

- userlogin/internal/middleware/taggingmiddleware.go
- userlogin/internal/middleware/versionmiddleware.go

可以看到路由文件中userlogin/internal/handler/routes.go 新增了一下代码
```go
server.AddRoutes(
		rest.WithMiddlewares(
			[]rest.Middleware{serverCtx.Tagging, serverCtx.Version},
			[]rest.Route{
				{
					Method:  http.MethodGet,
					Path:    "/api/tags",
					Handler: TagsHandler(serverCtx),
				},
			}...,
		),
	)
```

3、 文件中添加中间件依赖  userlogin/internal/svc/servicecontext.go 

```go
type ServiceContext struct {
    Config    config.Config
    Tagging   rest.Middleware
    Version   rest.Middleware
    UserModel user.UserModel
}

func NewServiceContext(c config.Config) *ServiceContext {
    conn := sqlx.NewMysql(c.Mysql.DataSource)
    
    return &ServiceContext{
        Config:    c,
        UserModel: user.NewUserModel(conn, c.CacheRedis),
        Tagging:   middleware.NewTaggingMiddleware().Handle,
        Version:   middleware.NewVersionMiddleware().Handle,
    }
}
```

4、启动测试
> http://127.0.0.1:8000/api/tags
> {     "tag": "tagV111--v1.1.0" }


## 全局中间件
userlogin/userlogin.go
```go
flag.Parse()

	var c config.Config
	conf.MustLoad(*configFile, &c)
	logx.MustSetup(c.LogConf)
	server := rest.MustNewServer(c.RestConf)
	defer server.Stop()
    
    // 全局中间件
	server.Use(func(next http.HandlerFunc) http.HandlerFunc {
		return func(writer http.ResponseWriter, request *http.Request) {

			logx.Info("global middleware")
			next(writer, request)
		}
	})


	ctx := svc.NewServiceContext(c)
	handler.RegisterHandlers(server, ctx)

	fmt.Printf("Starting server at %s:%d...\n", c.Host, c.Port)
	server.Start()
```


# **链路追踪**
> go-zero 框架已实现了链路追踪，并且Jaeger， Zipkin 这两种链路追踪上报工具，只要简单配置就实现链路追踪。


1、编辑配置文件 增加jaeger配置

```yaml
Telemetry:
    Name: user.api
    Endpoint: http://jaeger:14268/api/traces
    Sampler: 1.0
    Batcher: jaeger
```
2、userlogin/internal/config/config.go 增加相应的映射
```go
type Config struct {
    Telemetry trace.Config
}
```

3、启动测试
访问 jaeger UI界面 [http://127.0.0.1:5000/search](http://127.0.0.1:5000/search)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/12487795/1658583864578-174aebf0-f0b2-4fee-8ca4-0f2eedac5c2e.png#clientId=uc124a46a-374c-4&from=paste&height=786&id=u33955d93&originHeight=786&originWidth=1905&originalType=binary&ratio=1&rotation=0&showTitle=false&size=56200&status=done&style=none&taskId=ucd71d1c1-4c8a-4508-a9f8-1befe93faa0&title=&width=1905)

# 错误处理
> 在业务中还会定义一些业务性错误，常用做法都是通过 code、msg 两个字段来进行业务处理结果描述 


1. 自定义错误 userlogin/common/errorx/baseerror.go
```go
package errorx

const defaultCode = 1001
const DBErrorCode = 5001

type CodeError struct {
	Code int    `json:"code"`
	Msg  string `json:"msg"`
}

type CodeErrorResponse struct {
	Code int    `json:"code"`
	Msg  string `json:"msg"`
}

func NewCodeError(code int, msg string) error {
	return &CodeError{Code: code, Msg: msg}
}

func NewDefaultError(msg string) error {
	return NewCodeError(defaultCode, msg)
}

func (e *CodeError) Error() string {
	return e.Msg
}

func (e *CodeError) Data() *CodeErrorResponse {
	return &CodeErrorResponse{
		Code: e.Code,
		Msg:  e.Msg,
	}
}

```
2、业务逻辑代码中替换为自定义错误 userlogin/internal/logic/loginlogic.go
```go
if err != nil {
		if err == user.ErrNotFound {
			return nil, errorx.NewDefaultError("用户不存在")
		}
		return nil, errorx.NewCodeError(errorx.DBErrorCode, err.Error())
	}
```
3、开启自定义错误 userlogin/userlogin.go
```go
httpx.SetErrorHandler(func(err error) (int, interface{}) {
		switch e := err.(type) {
		case *errorx.CodeError:
			return http.StatusOK, e.Data()
		default:
			return http.StatusInternalServerError, nil
		}
	})
```
4、启动测试

```go
#  curl -i -X POST http://127.0.0.1:8000/api/login -H 'Content-Type: application/json' -d '{"email":"notfound@gmail.com","password":"****"}'
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Traceparent: 00-d004fc85afc1bcb7f4cf6218de12e518-5eed79e06803e434-01
Date: Mon, 25 Jul 2022 14:07:02 GMT
Content-Length: 37

{"code":1001,"msg":"用户不存在"}
```

自定义认证错误
> curl -i -X GET   '[http://127.0.0.1:8000/api/userinfo'](http://127.0.0.1:8000/api/userinfo')

```go
HTTP/1.1 401 Unauthorized
Traceparent: 00-f1a3a6dee278d0aa8604c5eab2c276a3-f4a8de7774b93cf4-01
Date: Mon, 25 Jul 2022 14:11:32 GMT
Content-Length: 0

```

修改代码 userlogin/userlogin.go
```go
unauthorized := rest.WithUnauthorizedCallback(func(w http.ResponseWriter, r *http.Request, err error) {
		httpx.WriteJson(w, http.StatusOK, errorx.NewCodeError(http.StatusUnauthorized, err.Error()))
	})
	server := rest.MustNewServer(c.RestConf, unauthorized)
```
测试
```go
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Traceparent: 00-f136c8b772b717f394871d447c2ad0af-400e2c6cd45ab383-01
Date: Mon, 25 Jul 2022 14:09:46 GMT
Content-Length: 48

{"code":401,"msg":"no token present in request"}


```
