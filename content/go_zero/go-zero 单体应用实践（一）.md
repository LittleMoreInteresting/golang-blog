+++
title = "go-zero 单体应用实践（一）"
description = "单体应用实践"
weight = 1
url = "/go-zero/01.html"
+++
# 环境搭建
> [官方文档](https://go-zero.dev/cn/docs/introduction)

- Golang 环境安装 👉 [golang](https://golang.google.cn/doc/install)
- Go Module设置 ` go env -w GO111MODULE="on" `
- goctl安装  👉 [goctl](https://go-zero.dev/cn/docs/goctl/installation)
- protoc & protoc-gen-go安装  ` goctl env check -i -f --verbose `

[etcd](https://etcd.io/docs/v3.5/)，[redis](https://redis.io/)，[mysql](https://www.mysql.com/) 等开发工具可以通过Docker 快速搭建；可参考👉 [gonivinck](https://github.com/nivin-studio/gonivinck)

# 创建单体应用
## 1、创建目录
```shell
mkdir user-login
cd user-login
go mod init user-login
```
## 2、编辑api文件 userlogin.api
[api语法介绍](https://go-zero.dev/cn/docs/design/grammar)
```shell
type (
	RegisterRequest {
		Name     string `json:"name"`
		Email    string `json:"email"`
		Password string `json:"password"`
	}

	RegisterResponse {
		ID    int    `json:"id"`
		Name  string `json:"name"`
		Email string `json:"email"`
	}

	LoginRequest {
		Email    string `json:"email"`
		Password string `json:"password"`
	}

	LoginResponse {
		Token  string `json:"token"`
		Expire int64  `json:"expire"`
	}
)

service userlogin-api {
	@handler RegisterHandler
	post /api/register(RegisterRequest) returns (RegisterResponse);
	
	@handler LoginHandler
	post /api/login(LoginRequest) returns (LoginResponse);
}
```
## 3、执行生成代码命令
> goctl api go -api userlogin.api -dir .
> go mod tidy 

生成文件目录
```shell
userlogin
├─etc // // 配置文件
├─internal
│  ├─config  // 配置声明type
│  ├─handler  // 路由及handler转发
│  ├─logic  // 业务逻辑
│  ├─svc // logic所依赖的资源池
│  └─types  // request、response的struct，根据api自动生成

```

4、编辑配置文件  userlogin/etc/userlogin-api.yaml
```yaml
Name: userlogin-api
Host: 0.0.0.0
Port: 8888
Mysql:
  DataSource: root:123456@tcp(mysql:3306)/user?charset=utf8mb4&parseTime=true&loc=Asia%2FShanghai

CacheRedis:
  - Host: redis:6379
    Type: node

Salt: DWe7OZf6KPlnv7yy
Auth:
  AccessSecret: uOvKLmVfztaXGpNYd4Z0I1SiT7MweJhl
  AccessExpire: 86400
```
添加config声明 userlogin/internal/config/config.go
其中Salt 用来密码加密存储，Auth  用于 [jwt鉴权](https://go-zero.dev/cn/docs/advance/jwt)
```go
type Config struct {
	rest.RestConf
	Mysql struct {
		DataSource string
	}

	CacheRedis cache.CacheConf
	Salt       string
	Auth       struct {
		AccessSecret string
		AccessExpire int64
	}
}

```

## 4、生成Model文件
添加用户表  userlogin/model/user.sql
```sql
CREATE TABLE `user` (
  `id` bigint unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(255)  NOT NULL DEFAULT '' COMMENT '用户姓名',
  `email` varchar(255)  NOT NULL DEFAULT '' COMMENT '用户邮箱',
  `password` varchar(255)  NOT NULL DEFAULT '' COMMENT '用户密码',
  `create_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_email_unique` (`email`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8mb4;
```
通过命令生成model文件 [model 指令](https://go-zero.dev/cn/docs/goctl/model)
> goctl model mysql ddl -src="userlogin/model/user.sql" -dir="userlogin/model/"  -c

-c 参数代表 在model层使用缓存 需要配置对应的 CacheRedis
生成文件如下：
```
user.sql
usermodel.go
usermodel_gen.go
vars.go
```
可以看到 goctl 已经生成了 UserModel interface 并实现了基本的增删改查询代码；
![image.png](https://cdn.nlark.com/yuque/0/2022/png/12487795/1657869696418-c471ff4c-d26d-4204-9429-55ec0921b534.png#clientId=ued4bc466-96bb-4&from=paste&height=524&id=uda9f261e&originHeight=524&originWidth=684&originalType=binary&ratio=1&rotation=0&showTitle=false&size=63053&status=done&style=none&taskId=u140ad78d-cb24-4458-81ea-e4cb522a9ee&title=&width=684)

## 5、完善服务依赖
编辑 userlogin/internal/svc/servicecontext.go 添加 logic所依赖的 UserModel
```go
type ServiceContext struct {
	Config config.Config

	UserModel model.UserModel
}

func NewServiceContext(c config.Config) *ServiceContext {
	conn := sqlx.NewMysql(c.Mysql.DataSource)

	return &ServiceContext{
		Config:    c,
		UserModel: model.NewUserModel(conn, c.CacheRedis),
	}
}
```

## 6、业务逻辑代码
1、注册逻辑代码
```go
type RegisterLogic struct {
	logx.Logger
	ctx    context.Context
	svcCtx *svc.ServiceContext
}

func NewRegisterLogic(ctx context.Context, svcCtx *svc.ServiceContext) *RegisterLogic {
	return &RegisterLogic{
		Logger: logx.WithContext(ctx),
		ctx:    ctx,
		svcCtx: svcCtx,
	}
}

func (l *RegisterLogic) Register(req *types.RegisterRequest) (resp *types.RegisterResponse, err error) {

	_, err = l.svcCtx.UserModel.FindOneByEmail(l.ctx, req.Email)
	if err == nil {
		return nil, status.Error(100, "该用户已存在")
	}
	if err != model.ErrNotFound {
		return nil, status.Error(100, err.Error())
	}
	newUser := model.User{
		Name:     req.Name,
		Email:    req.Email,
		Password: cryptx.PasswordEncrypt(l.svcCtx.Config.Salt, req.Password),
	}

	res, err := l.svcCtx.UserModel.Insert(l.ctx, &newUser)
	if err != nil {
		return nil, status.Error(500, err.Error())
	}

	newUser.Id, err = res.LastInsertId()
	if err != nil {
		return nil, status.Error(500, err.Error())
	}

	return &types.RegisterResponse{
		ID:    int(newUser.Id),
		Name:  newUser.Name,
		Email: newUser.Email,
	}, nil
}
```

2、添加 JWT 工具   userlogin/common/jwtx/jwt.go
```go
package jwtx

import "github.com/golang-jwt/jwt"

func GetToken(secretKey string, iat, seconds, uid int64) (string, error) {
	claims := make(jwt.MapClaims)
	claims["exp"] = iat + seconds
	claims["iat"] = iat
	claims["uid"] = uid
	token := jwt.New(jwt.SigningMethodHS256)
	token.Claims = claims
	return token.SignedString([]byte(secretKey))
}

```

3、用户登录逻辑  userlogin/internal/logic/loginlogic.go
```go
type LoginLogic struct {
	logx.Logger
	ctx    context.Context
	svcCtx *svc.ServiceContext
}

func NewLoginLogic(ctx context.Context, svcCtx *svc.ServiceContext) *LoginLogic {
	return &LoginLogic{
		Logger: logx.WithContext(ctx),
		ctx:    ctx,
		svcCtx: svcCtx,
	}
}

func (l *LoginLogic) Login(req *types.LoginRequest) (resp *types.LoginResponse, err error) {
	res, err := l.svcCtx.UserModel.FindOneByEmail(l.ctx, req.Email)
	if err != nil {
		if err == model.ErrNotFound {
			return nil, status.Error(100, "用户不存在")
		}
		return nil, status.Error(500, err.Error())
	}
	// 判断密码是否正确
	password := cryptx.PasswordEncrypt(l.svcCtx.Config.Salt, req.Password)
	if password != res.Password {
		return nil, status.Error(100, "密码错误")
	}
	now := time.Now().Unix()
	accessExpire := l.svcCtx.Config.Auth.AccessExpire

	accessToken, err := jwtx.GetToken(l.svcCtx.Config.Auth.AccessSecret, now, accessExpire, res.Id)
	if err != nil {
		return nil, err
	}
	return &types.LoginResponse{
		Token:  accessToken,
		Expire: now + accessExpire,
	}, nil
}
```

7、启动测试
```go
go run userlogin.go -f etc/userlogin-api.yaml
# Starting server at 0.0.0.0:8888...



```

注册测试
```shell
curl -i -X POST http://127.0.0.1:8888/api/register -H 'Content-Type: application/json' -d '{"name":"test","email":"test@gmail.com","password":"123456"}'

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Traceparent: 00-91ee4162c2a42c7df3111ab73cdb242f-1b886700e9099cc8-00
Date: Tue, 19 Jul 2022 16:19:33 GMT
Content-Length: 47

{"id":1,"name":"test","email":"test@gmail.com"}
```

登录测试
```shell
curl -i -X POST http://127.0.0.1:8888/api/login -H 'Content-Type: application/json' -d '{"email":"test@gmail.com","password":"123456"}'

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Traceparent: 00-4cc2b11a99a2216590c50a2a7215e832-d7af5dc45fa6a65b-00
Date: Tue, 19 Jul 2022 16:22:28 GMT
Content-Length: 171

{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2NTgzMzQxNDgsImlhdCI6MTY1ODI0Nzc0OCwidWlkIjoxfQ.QGrhGWCRSMWGyEZyV7GatGYU_XgBVdjQRJqW7qEBk04","expire":1658334148}
```


# jwt鉴权应用
上面已经完成了登录注册功能；并且登录返回了Token；接下来就用一下Token
## 1、开启jwt鉴权
编写 api 文件 在 service 上方声明使用jwt鉴权 
修改 userlogin/userlogin.api 重新执行 goctl api 命令
```go
type UserInfoResponse {
		ID    int64    `json:"id"`
		Name  string `json:"name"`
		Email string `json:"email"`
	}

@server(
	jwt: Auth // Auth 与 userlogin/internal/config/config.go 中配置的 jwt 参数对应
)
service userlogin-api {
	@handler UserInfo
	get /api/userinfo returns (UserInfoResponse)
}
```
可以看到路由文件中 userlogin/internal/handler/routes.go 新增的路由中通 WithJwt 声明了使用jwt鉴权
```go
server.AddRoutes(
		[]rest.Route{
			{
				Method:  http.MethodGet,
				Path:    "/api/userinfo",
				Handler: UserInfoHandler(serverCtx),
			},
		},
		rest.WithJwt(serverCtx.Config.Auth.AccessSecret),
	)
```
## 2、编写logic 代码
> userlogin/internal/logic/userinfologic.go

```go
func NewUserInfoLogic(ctx context.Context, svcCtx *svc.ServiceContext) *UserInfoLogic {
	return &UserInfoLogic{
		Logger: logx.WithContext(ctx),
		ctx:    ctx,
		svcCtx: svcCtx,
	}
}

func (l *UserInfoLogic) UserInfo() (resp *types.UserInfoResponse, err error) {
	uid, _ := l.ctx.Value("uid").(json.Number).Int64()
	one, err := l.svcCtx.UserModel.FindOne(l.ctx, uid)
	if err != nil {
		return nil, err
	}
	return &types.UserInfoResponse{
		ID:    one.Id,
		Name:  one.Name,
		Email: one.Email,
	}, nil
}
```
## 3、运行测试

```shell
# curl -i http://127.0.0.1:8000/api/userinfo

HTTP/1.1 401 Unauthorized
Traceparent: 00-a418d68212d9e2c29664a7e9aeefbd14-7282c005bbcd5f24-00
Date: Thu, 21 Jul 2022 17:01:21 GMT
Content-Length: 0

#curl -i -X GET  'http://127.0.0.1:8000/api/userinfo'  -H 'Authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2NTg1MDkzNDAsImlhdCI6MTY1ODQyMjk0MCwidWlkIjoxfQ.j6K0CjJ9jM5a3a7-DRUGq6b3uvMFcelvR7vQkZAQXWw'

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Traceparent: 00-e547b466d3e7da54d932838bfdee17c4-4ec7aaedb056ef03-00
Date: Thu, 21 Jul 2022 17:07:42 GMT
Content-Length: 47

{"id":1,"name":"test","email":"test@gmail.com"}

```



