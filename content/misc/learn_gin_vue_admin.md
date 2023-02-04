---
title: "gin-vue-admin 代码学习"
date: 2023-02-04T20:42:10+08:00
draft: false
---

# gin-vue-admin 项目学习

## 项目结构

```
├── deploy： 部署相关的文件
├── docs： 项目文档
├── server： 后端代码
└── web： 前端代码
```

项目分为两个部分，使用 vue 框架的前端部分和使用 gin 框架的后端部分，前端部分在`web` 目录下，后端部分在`server`目录下。我们主要关注 server 后端部分。了解一下目前 github 排名第一的 golang admin 项目是怎么设计的。

## 后端结构

```
├── api
│   └── v1: v1 版本的 api，相当于java中的 controller
├── config: 配置类，包括数据库配置，jwt 配置，redis 配置等
├── core
│   └── internal: 服务启动方法
├── docs: swagger 文档
├── global: 全局对象
├── initialize
│   └── internal: 初始化方法，日志，数据库，redis，jwt，定时任务等
├── middleware: 中间件,包括跨域，jwt，casbin，日志，全局异常处理等,使用了 gin.handleFunc 的特性
├── model: 数据库模型，请求参数，响应参数
│   ├── common
│   └── system
├── packfile
├── plugin:
│   ├── email: email 插件，是个完整的小项目，里面结构和外部类似
│   ├── plugin-tool:
│   └── ws
├── resource: 资源文件，包括自动生成代码的模板，页面模板，插件模板等
│   ├── autocode_template: 代码模板，从结构体生成代码
│   ├── page: 前端静态资源
│   └── plug_template: 插件代码模板
├── router:路由代码
│   └── system:
├── service: 业务代码，相当于 java 中的 service
│   └── system
├── source: 数据库的初始数据
│   └── system
└── utils: 公共的工具类
    ├── captcha
    ├── plugin
    ├── timer
    └── upload
```

## 项目启动

```go
func main() {
	global.GVA_VP = core.Viper() // 初始化Viper
	initialize.OtherInit()
	global.GVA_LOG = core.Zap() // 初始化zap日志库
	zap.ReplaceGlobals(global.GVA_LOG)
	global.GVA_DB = initialize.Gorm() // gorm连接数据库
	initialize.Timer()
	initialize.DBList()
	if global.GVA_DB != nil {
		initialize.RegisterTables(global.GVA_DB) // 初始化表
		// 程序结束前关闭数据库链接
		db, _ := global.GVA_DB.DB()
		defer db.Close()
	}
	core.RunWindowsServer()
}
```
