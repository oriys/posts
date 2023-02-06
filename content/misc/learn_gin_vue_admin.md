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
