+++
title = "go-zero 微服务应用实践（一）"
description = "单体应用实践"
weight = 4
url = "/go-zero/04.html"
+++
# 简介
> go-zero 是一个集成了各种工程实践的 web 和 rpc 框架。通过弹性设计保障了大并发服务端的稳定性，经受了充分的实战检验。

本节将用go-zero 开发一个用户服务；主要包括注册、登录、用户信息查询功能
# 初始化项目
```shell
//创建目录
madir project && cd project
//初始化项目
go mod init project
```

# 用户服务
0、环境准备 
[protoc & protoc-gen-go安装](https://go-zero.dev/cn/docs/prepare/protoc-install)
> protoc是一款用C++编写的工具，其可以将proto文件翻译为指定语言的代码。在go-zero的微服务中，我们采用grpc进行服务间的通信，而grpc的编写就需要用到protoc和翻译成go语言rpc stub代码的插件protoc-gen-go。


1、创建proto 文件 user/rpc/user.proto
```protobuf
syntax = "proto3";

package user;

option go_package = "./user";

message RegisterRequest {
  string Name = 1;
  int64 Gender = 2;
  string Email = 3;
  string Password = 4;
}

message RegisterResponse {
  int64 Id = 1;
  string Name = 2;
  int64 Gender = 3;
  string Email = 4;
}

message LoginRequest {
  string Email = 1;
  string Password = 2;
}
message LoginResponse {
  int64 Id = 1;
  string Name = 2;
  int64 Gender = 3;
  string Email = 4;
}

message UserInfoRequest {
  int64 Id = 1;
}
message UserInfoResponse {
  int64 Id = 1;
  string Name = 2;
  int64 Gender = 3;
  string Email = 4;
}

service User {
  rpc Register(RegisterRequest) returns(RegisterResponse);
  rpc Login(LoginRequest) returns(LoginResponse);
  rpc UserInfo(UserInfoRequest) returns(UserInfoResponse);
}
```

2、创建 user/generate.go
为了方便快速执行 goctl 相关命令，创建generate.go 文件 写入生成命令
```go
package user

//Rpc
//go:generate goctl rpc protoc ./rpc/user.proto --go_out=./rpc/types --go-grpc_out=./rpc/types --zrpc_out=./rpc/
//go:generate goctl model mysql ddl -src ./model/user.sql -dir ./model -c
//go:generate goctl api go -api ./api/user.api -dir ./api
```
执行 goctl rpc protoc 命令生成代码；代码目录如下
```powershell
──rpc
    ├─etc
    ├─internal
    │  ├─config
    │  ├─logic
    │  ├─server
    │  └─svc
    ├─types
    │  └─user
    └─user
```

3、创建model/user.sql 并执行 命令  
> goctl model mysql ddl -src ./model/user.sql -dir ./model -c

```sql
CREATE TABLE `user` (
    `id` bigint unsigned NOT NULL AUTO_INCREMENT,
    `name` varchar(255)  NOT NULL DEFAULT '' COMMENT '用户姓名',
    `gender` tinyint(3) unsigned NOT NULL DEFAULT '0' COMMENT '用户性别',
    `email` varchar(255)  NOT NULL DEFAULT '' COMMENT '用户邮箱',
    `password` varchar(255)  NOT NULL DEFAULT '' COMMENT '用户密码',
    `create_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
    `update_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (`id`),
    UNIQUE KEY `idx_email_unique` (`email`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8mb4;
```

4、注册逻辑开发 user/rpc/internal/logic/registerlogic.go
```go
func (l *RegisterLogic) Register(in *user.RegisterRequest) (*user.RegisterResponse, error) {
	_, err := l.svcCtx.UserModel.FindOneByEmail(l.ctx, in.Email)
	if err == nil {
		return nil, status.Error(100, "该邮箱被已注册")
	}
	if err != model.ErrNotFound {
		return nil, status.Error(500, err.Error())
	}

	newUser := model.User{
		Name:     in.Name,
		Gender:   in.Gender,
		Email:    in.Email,
		Password: cryptx.PasswordEncrypt(l.svcCtx.Config.Salt, in.Password),
	}
	res, err := l.svcCtx.UserModel.Insert(l.ctx, &newUser)
	if err != nil {
		return nil, status.Error(500, err.Error())
	}
	newUser.Id, err = res.LastInsertId()
	if err != nil {
		return nil, status.Error(500, err.Error())
	}

	return &user.RegisterResponse{
		Id:     newUser.Id,
		Name:   newUser.Name,
		Gender: newUser.Gender,
		Email:  newUser.Email,
	}, nil
}
```
其中密码需要加密存储，加密方法实现：user/common/cryptx/crypt.go
```go
func PasswordEncrypt(salt, password string) string {
	dk, _ := scrypt.Key([]byte(password), []byte(salt), 32768, 8, 1, 32)
	return fmt.Sprintf("%x", string(dk))
}
```

5、登录逻辑开发：user/rpc/internal/logic/loginlogic.go
```go
func (l *LoginLogic) Login(in *user.LoginRequest) (*user.LoginResponse, error) {
	res, err := l.svcCtx.UserModel.FindOneByEmail(l.ctx, in.Email)
	if err != nil {
		if err == model.ErrNotFound {
			return nil, status.Error(100, "用户不存在")
		}
		return nil, status.Error(500, err.Error())
	}

	password := cryptx.PasswordEncrypt(l.svcCtx.Config.Salt, in.Password)
	if password != res.Password {
		return nil, status.Error(100, "密码错误")
	}
	return &user.LoginResponse{
		Id:     res.Id,
		Name:   res.Name,
		Gender: res.Gender,
		Email:  res.Email,
	}, nil
}
```

6、查询用户信息：user/rpc/internal/logic/userinfologic.go
```go
func (l *UserInfoLogic) UserInfo(in *user.UserInfoRequest) (*user.UserInfoResponse, error) {
	// 查询用户是否存在
	res, err := l.svcCtx.UserModel.FindOne(l.ctx, in.Id)
	if err != nil {
		if err == model.ErrNotFound {
			return nil, status.Error(100, "用户不存在")
		}
		return nil, status.Error(500, err.Error())
	}
	return &user.UserInfoResponse{
		Id:     res.Id,
		Name:   res.Name,
		Gender: res.Gender,
		Email:  res.Email,
	}, nil
}
```


7。启动测试  go run user.go -f etc/user.yaml
```shell
> grpcurl -plaintext 127.0.0.1:8080 list
grpc.health.v1.Health
grpc.reflection.v1alpha.ServerReflection
user.User


> grpcurl -plaintext 127.0.0.1:8080 list user.User
user.User.Login
user.User.Register
user.User.UserInfo


grpcurl -plaintext -d '{"Name":"gopher","Gender":"1","Email":"123@qq.com","Password":"Password"}' \
    127.0.0.1:8080 user.User/Register
```

测试用到了[grpcurl](https://github.com/fullstorydev/grpcurl) 需要配置 Mode: dev 或test  可以看到user.go 中Mode为dev或test是增加了注册反射服务代码
```shell
s := zrpc.MustNewServer(c.RpcServerConf, func(grpcServer *grpc.Server) {
		user.RegisterUserServer(grpcServer, svr)

		if c.Mode == service.DevMode || c.Mode == service.TestMode {
			reflection.Register(grpcServer)
		}
	})
```

