+++
title = "go-zero 微服务应用实践（二）"
description = "单体应用实践"
weight = 5
url = "/go-zero/05.html"
+++
# 服务调用
完成rpc访问功能后，还需要进行客户端调用，会用到goctl 的api功能；创建api目录编写api文件；api相关语法可以参考官方文档：[api语法介绍](https://go-zero.dev/cn/docs/design/grammar)
## 1、API文件编写  
 其中 登录注册增加了参数验证，使用 [validator](https://github.com/go-playground/validator) 包进行验证详细使用方法可以到 [github.com/go-playground/validator](https://github.com/go-playground/validator) 查看
```go
syntax = "v1"
type (
	LoginRequest {
		Email    string `json:"Email" validate:"required,email"`
		Password string `json:"password" validate:"required,gte=8"`
	}
	LoginResponse {
		AccessToken  string `json:"accessToken"`
		AccessExpire int64  `json:"accessExpire"`
	}
	RegisterRequest {
		Name     string `json:"name" validate:"required,gte=2,lte=20"`
		Gender   int64  `json:"gender" validate:"oneof=1 2"`
		Email    string `json:"Email" validate:"required,email"`
		Password string `json:"password" validate:"required,gte=8"`
	}
	RegisterResponse {
		Id     int64  `json:"id"`
		Name   string `json:"name"`
		Gender int64  `json:"gender"`
		Email  string `json:"Email"`
	}

	UserInfoRequest {
	}
	UserInfoResponse {
		Id     int64  `json:"id"`
		Name   string `json:"name"`
		Gender int64  `json:"gender"`
		Email  string `json:"Email"`
	}
)

service User {
	@handler Login
	post /api/user/login(LoginRequest) returns (LoginResponse)
	
	@handler Register
	post /api/user/register(RegisterRequest) returns (RegisterResponse)
}

@server(
	jwt: Auth
)
service User {
	@handler UserInfo
	post /api/user/userinfo(UserInfoRequest) returns (UserInfoResponse)
}
```
## 2、执行goctl命令
> goctl api go -api ./api/user.api -dir ./api

执行后生成目录如下
```shell
api
├─etc
└─internal
    ├─config
    ├─handler
    ├─logic
    ├─svc
    └─types
```
## 3、Service 层增加 UserRpc
```go
package svc

import (
	"github.com/zeromicro/go-zero/zrpc"
	"slowdown/user/api/internal/config"
	"slowdown/user/rpc/user"
)

type ServiceContext struct {
	Config config.Config

	UserRpc user.User
}

func NewServiceContext(c config.Config) *ServiceContext {
	return &ServiceContext{
		Config:  c,
		UserRpc: user.NewUser(zrpc.MustNewClient(c.UserRpc)),
	}
}

```

## 4、logic 业务逻辑层
注册逻辑
```go
func (l *RegisterLogic) Register(req *types.RegisterRequest) (resp *types.RegisterResponse, err error) {
	validate := validator.New()
	err = validate.Struct(req)
	if err != nil {
		return nil, err
	}
	res, err := l.svcCtx.UserRpc.Register(l.ctx, &user.RegisterRequest{
		Name:     req.Name,
		Email:    req.Email,
		Gender:   req.Gender,
		Password: req.Password,
	})
	if err != nil {
		return nil, err
	}

	return &types.RegisterResponse{
		Id:     res.Id,
		Name:   res.Name,
		Gender: res.Gender,
		Email:  res.Email,
	}, nil
}
```

登录逻辑
```go
func (l *LoginLogic) Login(req *types.LoginRequest) (resp *types.LoginResponse, err error) {
	res, err := l.svcCtx.UserRpc.Login(l.ctx, &user.LoginRequest{
		Email:    req.Email,
		Password: req.Password,
	})
	if err != nil {
		return nil, err
	}
	now := time.Now().Unix()

	accessToken, err := jwtx.GetToken(l.svcCtx.Config.Auth.AccessSecret, now, l.svcCtx.Config.Auth.AccessExpire, res.Id)
	if err != nil {
		return nil, err
	}
	return &types.LoginResponse{
		AccessExpire: l.svcCtx.Config.Auth.AccessExpire,
		AccessToken:  accessToken,
	}, nil
}

```

获取用户信息
```go
func (l *UserInfoLogic) UserInfo() (resp *types.UserInfoResponse, err error) {
	uid, _ := l.ctx.Value("uid").(json.Number).Int64()
	res, err := l.svcCtx.UserRpc.UserInfo(l.ctx, &user.UserInfoRequest{
		Id: uid,
	})
	if err != nil {
		return nil, err
	}
	return &types.UserInfoResponse{
		Id:     res.Id,
		Name:   res.Name,
		Gender: res.Gender,
		Email:  res.Email,
	}, nil
}
```
## 5、启动服务测试
使用apifox进行测试 
![image.png](/img/go-zero/go-zero-05-01.png)
![image.png](/img/go-zero/go-zero-05-02.png)
