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

这段代码主要是初始化一些全局变量，比如数据库连接，日志，定时任务等。然后启动服务。

```go
func Viper(path ...string) *viper.Viper {
	var config string

	if len(path) == 0 {
		flag.StringVar(&config, "c", "", "choose config file.")
		flag.Parse()
		if config == "" { // 判断命令行参数是否为空
			if configEnv := os.Getenv(internal.ConfigEnv); configEnv == "" { // 判断 internal.ConfigEnv 常量存储的环境变量是否为空
				switch gin.Mode() {
				case gin.DebugMode:
					config = internal.ConfigDefaultFile
					fmt.Printf("您正在使用gin模式的%s环境名称,config的路径为%s\n", gin.EnvGinMode, internal.ConfigDefaultFile)
				case gin.ReleaseMode:
					config = internal.ConfigReleaseFile
					fmt.Printf("您正在使用gin模式的%s环境名称,config的路径为%s\n", gin.EnvGinMode, internal.ConfigReleaseFile)
				case gin.TestMode:
					config = internal.ConfigTestFile
					fmt.Printf("您正在使用gin模式的%s环境名称,config的路径为%s\n", gin.EnvGinMode, internal.ConfigTestFile)
				}
			} else { // internal.ConfigEnv 常量存储的环境变量不为空 将值赋值于config
				config = configEnv
				fmt.Printf("您正在使用%s环境变量,config的路径为%s\n", internal.ConfigEnv, config)
			}
		} else { // 命令行参数不为空 将值赋值于config
			fmt.Printf("您正在使用命令行的-c参数传递的值,config的路径为%s\n", config)
		}
	} else { // 函数传递的可变参数的第一个值赋值于config
		config = path[0]
		fmt.Printf("您正在使用func Viper()传递的值,config的路径为%s\n", config)
	}

	v := viper.New()
	v.SetConfigFile(config)
	v.SetConfigType("yaml")
	err := v.ReadInConfig()
	if err != nil {
		panic(fmt.Errorf("Fatal error config file: %s \n", err))
	}
	v.WatchConfig()

	v.OnConfigChange(func(e fsnotify.Event) {
		fmt.Println("config file changed:", e.Name)
		if err = v.Unmarshal(&global.GVA_CONFIG); err != nil {
			fmt.Println(err)
		}
	})
	if err = v.Unmarshal(&global.GVA_CONFIG); err != nil {
		fmt.Println(err)
	}

	// root 适配性 根据root位置去找到对应迁移位置,保证root路径有效
	global.GVA_CONFIG.AutoCode.Root, _ = filepath.Abs("..")
	return v
}
```

`viper` 是 `spf13/viper` 开发的一个配置文件解析包，可以解析 `json`，`toml`，`yaml`，`hcl`，`env` 等多种格式的配置文件。gin-vue-admin 项目使用了 viper 来解析配置文件，配置文件的路径在`server/config.yaml`。

在`core.Viper()`中，首先判断是否有命令行参数传递，如果没有，再判断是否有环境变量传递，根据 `gin` 的运行模式，选择对应的配置文件路径，如果都没有，就使用默认的配置文件路径。

里面用到了 viper 的几个方法

`v.SetConfigFile(config)`：设置配置文件的内容  
`v.SetConfigType("yaml")`：设置配置文件的格式  
`v.ReadInConfig()`：解析配置文件  
`v.WatchConfig()`：监听配置文件的变化  
`v.OnConfigChange(func)`：配置文件发生变化时的回调函数

```go
func OtherInit() {
	dr, err := utils.ParseDuration(global.GVA_CONFIG.JWT.ExpiresTime)
	if err != nil {
		panic(err)
	}
	_, err = utils.ParseDuration(global.GVA_CONFIG.JWT.BufferTime)
	if err != nil {
		panic(err)
	}

	global.BlackCache = local_cache.NewCache(
		local_cache.SetDefaultExpire(dr),
	)
}
```

`OtherInit()`函数中，主要是对一些其他的初始化操作，比如 `jwt` 的过期时间，`jwt` 的缓冲时间，`jwt` 的黑名单缓存。

```go
func Zap() (logger *zap.Logger) {
	if ok, _ := utils.PathExists(global.GVA_CONFIG.Zap.Director); !ok { // 判断是否有Director文件夹
		fmt.Printf("create %v directory\n", global.GVA_CONFIG.Zap.Director)
		_ = os.Mkdir(global.GVA_CONFIG.Zap.Director, os.ModePerm)
	}

	cores := internal.Zap.GetZapCores()
	logger = zap.New(zapcore.NewTee(cores...))

	if global.GVA_CONFIG.Zap.ShowLine {
		logger = logger.WithOptions(zap.AddCaller())
	}
	return logger
}
```

`Zap()`函数中，主要是对 `zap` 的初始化操作，`zap` 是 `uber-go/zap` 开发的一个日志库，可以将日志输出到控制台，文件，`kafka` 等多种地方。

```go
func Gorm() *gorm.DB {
	switch global.GVA_CONFIG.System.DbType {
	case "mysql":
		return GormMysql()
	case "pgsql":
		return GormPgSql()
	case "oracle":
		return GormOracle()
  case "mssql":
		return GormMssql()
	default:
		return GormMysql()
	}
}
```

`Gorm()`函数中，主要是对 `gorm` 的初始化操作，`gorm` 是 `jinzhu/gorm` 开发的一个 `orm` 库，可以对 `mysql`，`pgsql`，`oracle`，`mssql` 等多种数据库进行操作。这段代码中，根据配置文件中的 `DbType` 字段，选择对应的数据库进行初始化。

以`MySql`为例

```go
func GormMysql() *gorm.DB {
	m := global.GVA_CONFIG.Mysql
	if m.Dbname == "" {
		return nil
	}
	mysqlConfig := mysql.Config{
		DSN:                       m.Dsn(), // DSN data source name
		DefaultStringSize:         191,     // string 类型字段的默认长度
		SkipInitializeWithVersion: false,   // 根据版本自动配置

	}
	if db, err := gorm.Open(mysql.New(mysqlConfig), internal.Gorm.Config(m.Prefix, m.Singular)); err != nil {
		return nil
	} else {
		db.InstanceSet("gorm:table_options", "ENGINE="+m.Engine)
		sqlDB, _ := db.DB()
		sqlDB.SetMaxIdleConns(m.MaxIdleConns)
		sqlDB.SetMaxOpenConns(m.MaxOpenConns)
		return db
	}
}
```

`GormMysql()`函数中，主要是对 `mysql` 的初始化操作，`mysql` 是 `go-sql-driver/mysql` 开发的一个 `mysql` 驱动库，可以对 `mysql` 进行操作。这段代码中，根据配置文件中的 `Mysql` 字段，对 `mysql` 进行初始化。

```go
func Timer() {
	if global.GVA_CONFIG.Timer.Start {
		for i := range global.GVA_CONFIG.Timer.Detail {
			go func(detail config.Detail) {
				var option []cron.Option
				if global.GVA_CONFIG.Timer.WithSeconds {
					option = append(option, cron.WithSeconds())
				}
				_, err := global.GVA_Timer.AddTaskByFunc("ClearDB", global.GVA_CONFIG.Timer.Spec, func() {
					err := utils.ClearTable(global.GVA_DB, detail.TableName, detail.CompareField, detail.Interval)
					if err != nil {
						fmt.Println("timer error:", err)
					}
				}, option...)
				if err != nil {
					fmt.Println("add timer error:", err)
				}
			}(global.GVA_CONFIG.Timer.Detail[i])
		}
	}
}
```

`Timer()`函数中，主要是对定时任务的初始化操作，`cron` 是 `robfig/cron` 开发的一个定时任务库，可以对定时任务进行操作。这段代码中，根据配置文件中的 `Timer` 字段，对定时任务进行初始化。

```go
func DBList() {
	dbMap := make(map[string]*gorm.DB)
	for _, info := range global.GVA_CONFIG.DBList {
		if info.Disable {
			continue
		}
		switch info.Type {
		case "mysql":
			dbMap[info.AliasName] = GormMysqlByConfig(config.Mysql{GeneralDB: info.GeneralDB})
		case "mssql":
			dbMap[info.AliasName] = GormMssqlByConfig(config.Mssql{GeneralDB: info.GeneralDB})
		case "pgsql":
			dbMap[info.AliasName] = GormPgSqlByConfig(config.Pgsql{GeneralDB: info.GeneralDB})
		case "oracle":
			dbMap[info.AliasName] = GormOracleByConfig(config.Oracle{GeneralDB: info.GeneralDB})
		default:
			continue
		}
	}
	// 做特殊判断,是否有迁移
	// 适配低版本迁移多数据库版本
	if sysDB, ok := dbMap[sys]; ok {
		global.GVA_DB = sysDB
	}
	global.GVA_DBList = dbMap
}
```

`DBList()`函数中，主要是对多数据库的初始化操作，这段代码中，根据配置文件中的 `DBList` 字段，对多数据库进行初始化。


```go
func RunWindowsServer() {
	if global.GVA_CONFIG.System.UseMultipoint || global.GVA_CONFIG.System.UseRedis {
		// 初始化redis服务
		initialize.Redis()
	}

	// 从db加载jwt数据
	if global.GVA_DB != nil {
		system.LoadAll()
	}

	Router := initialize.Routers()
	Router.Static("/form-generator", "./resource/page")

	address := fmt.Sprintf(":%d", global.GVA_CONFIG.System.Addr)
	s := initServer(address, Router)
	// 保证文本顺序输出
	// In order to ensure that the text order output can be deleted
	time.Sleep(10 * time.Microsecond)
	global.GVA_LOG.Info("server run success on ", zap.String("address", address))

	fmt.Printf(`
	欢迎使用 gin-vue-admin
	当前版本:v2.5.5
    加群方式:微信号：shouzi_1994 QQ群：622360840
	插件市场:https://plugin.gin-vue-admin.com
	GVA讨论社区:https://support.qq.com/products/371961
	默认自动化文档地址:http://127.0.0.1%s/swagger/index.html
	默认前端文件运行地址:http://127.0.0.1:8080
	如果项目让您获得了收益，希望您能请团队喝杯可乐:https://www.gin-vue-admin.com/coffee/index.html
`, address)
	global.GVA_LOG.Error(s.ListenAndServe().Error())
}
```

`RunWindowsServer()`函数中，主要是对服务的初始化操作，这段代码中，根据配置文件中的 `System` 字段，对服务进行初始化。


## 完成一次请求的流程

### 接口定义

在`main()`函数中，里面有个`initialize.Routers()`函数，这个函数是用来初始化路由的，先来看看这个函数的定义。

```go
func Routers() *gin.Engine {
	Router := gin.Default()
	systemRouter := router.RouterGroupApp.System
	exampleRouter := router.RouterGroupApp.Example
	// 如果想要不使用nginx代理前端网页，可以修改 web/.env.production 下的
	// VUE_APP_BASE_API = /
	// VUE_APP_BASE_PATH = http://localhost
	// 然后执行打包命令 npm run build。在打开下面4行注释
	// Router.LoadHTMLGlob("./dist/*.html") // npm打包成dist的路径
	// Router.Static("/favicon.ico", "./dist/favicon.ico")
	// Router.Static("/static", "./dist/assets")   // dist里面的静态资源
	// Router.StaticFile("/", "./dist/index.html") // 前端网页入口页面

	Router.StaticFS(global.GVA_CONFIG.Local.Path, http.Dir(global.GVA_CONFIG.Local.StorePath)) // 为用户头像和文件提供静态地址
	// Router.Use(middleware.LoadTls())  // 如果需要使用https 请打开此中间件 然后前往 core/server.go 将启动模式 更变为 Router.RunTLS("端口","你的cre/pem文件","你的key文件")
	// 跨域，如需跨域可以打开下面的注释
	// Router.Use(middleware.Cors()) // 直接放行全部跨域请求
	// Router.Use(middleware.CorsByRules()) // 按照配置的规则放行跨域请求
	//global.GVA_LOG.Info("use middleware cors")
	docs.SwaggerInfo.BasePath = global.GVA_CONFIG.System.RouterPrefix
	Router.GET(global.GVA_CONFIG.System.RouterPrefix+"/swagger/*any", ginSwagger.WrapHandler(swaggerFiles.Handler))
	global.GVA_LOG.Info("register swagger handler")
	// 方便统一添加路由组前缀 多服务器上线使用

	PublicGroup := Router.Group(global.GVA_CONFIG.System.RouterPrefix)
	{
		// 健康监测
		PublicGroup.GET("/health", func(c *gin.Context) {
			c.JSON(http.StatusOK, "ok")
		})
	}
	{
		systemRouter.InitBaseRouter(PublicGroup) // 注册基础功能路由 不做鉴权
		systemRouter.InitInitRouter(PublicGroup) // 自动初始化相关
	}
	PrivateGroup := Router.Group(global.GVA_CONFIG.System.RouterPrefix)
	PrivateGroup.Use(middleware.JWTAuth()).Use(middleware.CasbinHandler())
	{
		systemRouter.InitApiRouter(PrivateGroup)                 // 注册功能api路由
		systemRouter.InitJwtRouter(PrivateGroup)                 // jwt相关路由
		systemRouter.InitUserRouter(PrivateGroup)                // 注册用户路由
		systemRouter.InitMenuRouter(PrivateGroup)                // 注册menu路由
		systemRouter.InitSystemRouter(PrivateGroup)              // system相关路由
		systemRouter.InitCasbinRouter(PrivateGroup)              // 权限相关路由
		systemRouter.InitAutoCodeRouter(PrivateGroup)            // 创建自动化代码
		systemRouter.InitAuthorityRouter(PrivateGroup)           // 注册角色路由
		systemRouter.InitSysDictionaryRouter(PrivateGroup)       // 字典管理
		systemRouter.InitAutoCodeHistoryRouter(PrivateGroup)     // 自动化代码历史
		systemRouter.InitSysOperationRecordRouter(PrivateGroup)  // 操作记录
		systemRouter.InitSysDictionaryDetailRouter(PrivateGroup) // 字典详情管理
		systemRouter.InitAuthorityBtnRouterRouter(PrivateGroup)  // 字典详情管理

		exampleRouter.InitCustomerRouter(PrivateGroup)              // 客户路由
		exampleRouter.InitFileUploadAndDownloadRouter(PrivateGroup) // 文件上传下载功能路由

		// Code generated by github.com/flipped-aurora/gin-vue-admin/server Begin; DO NOT EDIT.

		// Code generated by github.com/flipped-aurora/gin-vue-admin/server End; DO NOT EDIT.
	}

	InstallPlugin(Router) // 安装插件

	global.GVA_LOG.Info("router register success")
	return Router
}
```

这里的代码主要的作用是配置路由和中间件，web框架是gin。

`gin.Default`是gin的默认配置，包含了日志、错误恢复等一些配置。

`gin.StaticFS`是gin的静态文件服务，这里是为了提供用户头像和文件的静态地址。接受两个参数，第一个是静态文件的访问路径，第二个是静态文件的存储路径。

`ginSwagger.WrapHandler`是gin的swagger文档服务，这里是为了提供swagger文档的访问地址。接受一个参数，是swagger文档的访问路径。

`gin.Group`是gin的路由分组，这里是为了提供路由分组的功能。接受一个参数，是路由分组的前缀。返回一个`gin.RouterGroup`对象。这里创建了两个路由分组，一个是公共路由分组，一个是私有路由分组。公共路由分组不需要健全，里面包含了健康检查的接口。私有路由分组需要健全，里面包含了所有的业务接口。

`gin.RouterGroup.Use`是gin的中间件，这里是为了提供中间件的功能。接受一个或多个参数，是中间件的函数。返回一个`gin.IRoutes`对象。这里使用了两个中间件，一个是jwt中间件，一个是casbin中间件。


### 路由注册

选择一个路由查看路由是怎么注册的，比如`InitApiRouter`。

```go
type ApiRouter struct{}

func (s *ApiRouter) InitApiRouter(Router *gin.RouterGroup) {
	apiRouter := Router.Group("api").Use(middleware.OperationRecord())
	apiRouterWithoutRecord := Router.Group("api")
	apiRouterApi := v1.ApiGroupApp.SystemApiGroup.SystemApiApi
	{
		apiRouter.POST("createApi", apiRouterApi.CreateApi)               // 创建Api
		apiRouter.POST("deleteApi", apiRouterApi.DeleteApi)               // 删除Api
		apiRouter.POST("getApiById", apiRouterApi.GetApiById)             // 获取单条Api消息
		apiRouter.POST("updateApi", apiRouterApi.UpdateApi)               // 更新api
		apiRouter.DELETE("deleteApisByIds", apiRouterApi.DeleteApisByIds) // 删除选中api
	}
	{
		apiRouterWithoutRecord.POST("getAllApis", apiRouterApi.GetAllApis) // 获取所有api
		apiRouterWithoutRecord.POST("getApiList", apiRouterApi.GetApiList) // 获取Api列表
	}
}
```

这里的代码主要的作用是注册路由，这里的路由是`api`。在这个路由分组下又声明了两个路由分组，一个是需要记录操作日志的路由分组，一个是不需要记录操作日志的路由分组。这里的路由分组是为了提供路由分组的功能。接受一个参数，是路由分组的前缀。返回一个`gin.RouterGroup`对象。这里使用了一个记录操作日志的中间件。


### 中间件实现

在上文中使用`gin.use`来注册中间件，现在来看一下如何实现一个中间件。

```go
func OperationRecord() gin.HandlerFunc {
	return func(c *gin.Context) {
		var body []byte
		var userId int
		if c.Request.Method != http.MethodGet {
			var err error
			body, err = io.ReadAll(c.Request.Body)
			if err != nil {
				global.GVA_LOG.Error("read body from request error:", zap.Error(err))
			} else {
				c.Request.Body = io.NopCloser(bytes.NewBuffer(body))
			}
		} else {
			query := c.Request.URL.RawQuery
			query, _ = url.QueryUnescape(query)
			split := strings.Split(query, "&")
			m := make(map[string]string)
			for _, v := range split {
				kv := strings.Split(v, "=")
				if len(kv) == 2 {
					m[kv[0]] = kv[1]
				}
			}
			body, _ = json.Marshal(&m)
		}
		claims, _ := utils.GetClaims(c)
		if claims.ID != 0 {
			userId = int(claims.ID)
		} else {
			id, err := strconv.Atoi(c.Request.Header.Get("x-user-id"))
			if err != nil {
				userId = 0
			}
			userId = id
		}
		record := system.SysOperationRecord{
			Ip:     c.ClientIP(),
			Method: c.Request.Method,
			Path:   c.Request.URL.Path,
			Agent:  c.Request.UserAgent(),
			Body:   string(body),
			UserID: userId,
		}

		// 上传文件时候 中间件日志进行裁断操作
		if strings.Contains(c.GetHeader("Content-Type"), "multipart/form-data")  {
			if len(record.Body) > 1024 {
				// 截断
				newBody := respPool.Get().([]byte)
				copy(newBody, record.Body)
				record.Body = string(newBody)
				defer respPool.Put(newBody[:0])
			}
		}

		writer := responseBodyWriter{
			ResponseWriter: c.Writer,
			body:           &bytes.Buffer{},
		}
		c.Writer = writer
		now := time.Now()

		c.Next()

		latency := time.Since(now)
		record.ErrorMessage = c.Errors.ByType(gin.ErrorTypePrivate).String()
		record.Status = c.Writer.Status()
		record.Latency = latency
		record.Resp = writer.body.String()

		if strings.Contains(c.Writer.Header().Get("Pragma"), "public")  ||
			strings.Contains(c.Writer.Header().Get("Expires"), "0")  ||
			strings.Contains(c.Writer.Header().Get("Cache-Control"), "must-revalidate, post-check=0, pre-check=0") ||
			strings.Contains(c.Writer.Header().Get("Content-Type"), "application/force-download")  ||
			strings.Contains(c.Writer.Header().Get("Content-Type"), "application/octet-stream")  ||
			strings.Contains(c.Writer.Header().Get("Content-Type"), "application/vnd.ms-excel")  ||
			strings.Contains(c.Writer.Header().Get("Content-Type"), "application/download")  ||
			strings.Contains(c.Writer.Header().Get("Content-Disposition"), "attachment")  ||
			strings.Contains(c.Writer.Header().Get("Content-Transfer-Encoding"), "binary") {
			if len(record.Resp) > 1024 {
				// 截断
				newBody := respPool.Get().([]byte)
				copy(newBody, record.Resp)
				record.Body = string(newBody)
				defer respPool.Put(newBody[:0])
			}
		}

		if err := operationRecordService.CreateSysOperationRecord(record); err != nil {
			global.GVA_LOG.Error("create operation record error:", zap.Error(err))
		}
	}
}
```

在这段代码的实现中，返回了一个入参是`gin.Context`，返回值是`gin.HandlerFunc`的函数，这个函数的作用是在每次请求的时候都会执行，可以在这个函数中实现一些逻辑，比如记录日志，记录请求的参数，记录请求的响应等等。  
在可以看到这个函数中使用了`gin.Context`的`Next`方法，这个方法的作用是执行下一个中间件，如果没有下一个中间件，那么就会执行路由的处理函数。





### 请求实现

```go
func (s *SystemApiApi) CreateApi(c *gin.Context) {
	var api system.SysApi
	err := c.ShouldBindJSON(&api)
	if err != nil {
		response.FailWithMessage(err.Error(), c)
		return
	}
	err = utils.Verify(api, utils.ApiVerify)
	if err != nil {
		response.FailWithMessage(err.Error(), c)
		return
	}
	err = apiService.CreateApi(api)
	if err != nil {
		global.GVA_LOG.Error("创建失败!", zap.Error(err))
		response.FailWithMessage("创建失败", c)
		return
	}
	response.OkWithMessage("创建成功", c)
}
```

在这段代码中，我们可以看到，首先使用了`gin.Context`的`ShouldBindJSON`方法，这个方法的作用是将请求的参数绑定到结构体中，这个方法的入参是一个指针，这个指针指向的结构体就是我们要绑定的结构体，这个方法会将请求的参数绑定到这个结构体中，如果绑定失败，那么就会返回一个错误，这个错误就是绑定失败的原因。

在后面，使用使用了`utils.Verify`来检验入参数，方法接受两个参数，第一个参数是要检验的结构体，第二个参数是检验的规则。

```go
func Verify(st interface{}, roleMap Rules) (err error) {
	compareMap := map[string]bool{
		"lt": true,
		"le": true,
		"eq": true,
		"ne": true,
		"ge": true,
		"gt": true,
	}

	typ := reflect.TypeOf(st)
	val := reflect.ValueOf(st) // 获取reflect.Type类型

	kd := val.Kind() // 获取到st对应的类别
	if kd != reflect.Struct {
		return errors.New("expect struct")
	}
	num := val.NumField()
	// 遍历结构体的所有字段
	for i := 0; i < num; i++ {
		tagVal := typ.Field(i)
		val := val.Field(i)
		if tagVal.Type.Kind() == reflect.Struct {
			if err = Verify(val.Interface(), roleMap); err != nil {
				return err
			}
		}
		if len(roleMap[tagVal.Name]) > 0 {
			for _, v := range roleMap[tagVal.Name] {
				switch {
				case v == "notEmpty":
					if isBlank(val) {
						return errors.New(tagVal.Name + "值不能为空")
					}
				case strings.Split(v, "=")[0] == "regexp":
					if !regexpMatch(strings.Split(v, "=")[1], val.String()) {
						return errors.New(tagVal.Name + "格式校验不通过")
					}
				case compareMap[strings.Split(v, "=")[0]]:
					if !compareVerify(val, v) {
						return errors.New(tagVal.Name + "长度或值不在合法范围," + v)
					}
				}
			}
		}
	}
	return nil
}
```

在这段代码中，首先使用了`reflect`包中的`TypeOf`和`ValueOf`方法，这两个方法的作用是获取到结构体的类型和值，然后使用`Kind`方法获取到结构体的类型，如果类型不是结构体，那么就会返回一个错误，如果是结构体，那么就会获取到结构体的字段数量，然后遍历结构体的所有字段，如果字段的类型是结构体，那么就会递归调用`Verify`方法，如果字段的类型不是结构体，那么就会获取到字段的名称，然后判断这个字段的名称是否在检验规则中，如果在，那么就会遍历这个字段的所有检验规则，然后根据规则的不同，调用不同的方法来检验这个字段的值。

因为缺少了java的注解功能，所以校验单独卸载了一个文件中，下面是这个文件的一个内容。

```go
var (
	IdVerify               = Rules{"ID": []string{NotEmpty()}}
	ApiVerify              = Rules{"Path": {NotEmpty()}, "Description": {NotEmpty()}, "ApiGroup": {NotEmpty()}, "Method": {NotEmpty()}}
	MenuVerify             = Rules{"Path": {NotEmpty()}, "ParentId": {NotEmpty()}, "Name": {NotEmpty()}, "Component": {NotEmpty()}, "Sort": {Ge("0")}}
	MenuMetaVerify         = Rules{"Title": {NotEmpty()}}
	LoginVerify            = Rules{"CaptchaId": {NotEmpty()}, "Username": {NotEmpty()}, "Password": {NotEmpty()}}
	RegisterVerify         = Rules{"Username": {NotEmpty()}, "NickName": {NotEmpty()}, "Password": {NotEmpty()}, "AuthorityId": {NotEmpty()}}
	PageInfoVerify         = Rules{"Page": {NotEmpty()}, "PageSize": {NotEmpty()}}
	CustomerVerify         = Rules{"CustomerName": {NotEmpty()}, "CustomerPhoneData": {NotEmpty()}}
	AutoCodeVerify         = Rules{"Abbreviation": {NotEmpty()}, "StructName": {NotEmpty()}, "PackageName": {NotEmpty()}, "Fields": {NotEmpty()}}
	AutoPackageVerify      = Rules{"PackageName": {NotEmpty()}}
	AuthorityVerify        = Rules{"AuthorityId": {NotEmpty()}, "AuthorityName": {NotEmpty()}}
	AuthorityIdVerify      = Rules{"AuthorityId": {NotEmpty()}}
	OldAuthorityVerify     = Rules{"OldAuthorityId": {NotEmpty()}}
	ChangePasswordVerify   = Rules{"Password": {NotEmpty()}, "NewPassword": {NotEmpty()}}
	SetUserAuthorityVerify = Rules{"AuthorityId": {NotEmpty()}}
)
```

接下来就是执行service方法的部分,里面包含的DB操作使用了gorm框架，这里就不多说了，下面是一个service方法的例子。

```go
func (apiService *ApiService) CreateApi(api system.SysApi) (err error) {
	if !errors.Is(global.GVA_DB.Where("path = ? AND method = ?", api.Path, api.Method).First(&system.SysApi{}).Error, gorm.ErrRecordNotFound) {
		return errors.New("存在相同api")
	}
	return global.GVA_DB.Create(&api).Error
}
```

