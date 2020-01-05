
[TOC]

`GF`框架提供了完善的`WebServer`日志管理功能，包括`access log`以及`error log`，并可自定义日志输出回调方法，开发者可灵活使用。

# 相关配置
配置对象：
https://godoc.org/github.com/gogf/gf/net/ghttp#ServerConfig

## 配置属性
日志相关配置属性如下：
```go
Logger            *glog.Logger      // Logger for server.
LogPath           string            // Directory for storing logging files.
LogStdout         bool              // Printing logging content to stdout.
ErrorStack        bool              // Logging stack information when error.
ErrorLogEnabled   bool              // Enable error logging files.
ErrorLogPattern   string            // Error log file pattern like: error-{Ymd}.log
AccessLogEnabled  bool              // Enable access logging files.
AccessLogPattern  string            // Error log file pattern like: access-{Ymd}.log
```
简要说明：
1. 默认情况下，日志不会输出到文件中，而是直接打印到终端。默认情况下的access日志终端输出是关闭的，仅有error日志默认开启。
1. 所有的选项均可通过`Server.Set*`方法设置，大部分选项可以通过`Server.Get*`方法获取。
1. 日志管理的配置选项也可通过配置文件进行便捷的配置，具体请参考【[配置管理](net/ghttp/config.md)】章节。
1. `Logger`是一个自定义的日志管理对象，开发者也可以传递一个完整的日志管理对象，忽略其他日志选项配置。
1. `LogPath`属性用于设置日志目录，只有在设置了日志目录的情况下才会输出日志到日志文件中。
1. `ErrorLogPattern`及`AccessLogPattern`用于配置日志文件名称格式，默认为`error-{Ymd}.log`及`access-{Ymd}.log`，例如：`error-20191212.log`, `access-20191212.log`。
1. 其他配置选项说明请参考注释和模块API文档。

## 配置文件

一个参考的配置文件示例（以`toml`格式为例）：
```toml
[server]
	LogPath           = "/var/log/gf-demos/server"
    LogStdout         = false            
    ErrorStack        = true              
    ErrorLogEnabled   = true              
    ErrorLogPattern   = "error.{Ymd}.log"            
    AccessLogEnabled  = true        
    AccessLogPattern  = "access.{Ymd}.log"    
```

当`Server`启动时将会自动去读取默认配置文件`config.toml`中的`server`节点配置。

# 使用示例

配置文件的方式比较简单，这里不再示例说明。以下示例通过配置方法的方式进行对`Server`进行配置。

## 请求日志

我们来看一个简单的例子：
```go
package main

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/net/ghttp"
)

func main() {
    s := g.Server()
    s.BindHandler("/log/access", func(r *ghttp.Request){
        r.Response.Writeln("请在运行终端查看日志输出")
    })
    s.SetAccessLogEnabled(true)
    s.SetPort(8199)
    s.Run()
}
```

我们在任何地方(包括运行时)可以调用`SetAccessLogEnabled(true)`开启`access log`的记录功能(并可通过`SetAccessLogEnabled(false)`随时动态关闭日志记录功能)，默认情况下，日志内容将会输出到终端界面。如以上示例程序执行后，访问 http://127.0.0.1:8199/log/access ，日志内容将会输出到终端上，如下：
```shell
2018-04-20 18:11:57.344 200 "GET http 127.0.0.1:8199 /log/access HTTP/1.1" 0.120, 127.0.0.1, "", "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Ubuntu Chromium/53.0.2785.143 Chrome/53.0.2785.143 Safari/537.36"
```
日志格式: `访问时间(精确到毫秒)` `HTTP状态码` "`请求方式` `请求前缀` `请求地址` `请求协议`" `执行时间(毫秒)` `客户端IP` `来源URL` `UserAgent`。其中，`请求前缀`为`http`或者`https`，`请求协议`往往为`HTTP/1.0`或者`HTTP/1.1`。

> 注意，日志中记录的`执行时间`单位为`毫秒`，绝大多数情况下看到的时间几乎都是`0.xxx`毫秒时间，也就是说执行时间都是纳秒级不到1毫秒。

## 错误日志

`WebServer`运行产生的任何错误信息(默认开启，输出到终端)，都将会被记录到`error log`中，以下是一个手动触发异常的示例：

```go
package main

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/net/ghttp"
)

func main() {
    s := g.Server()
    s.BindHandler("/log/error", func(r *ghttp.Request){
        panic("OMG")
    })
    s.SetErrorLogEnabled(true)
    s.SetPort(8199)
    s.Run()
}
```

运行并访问后，终端输出的错误日志信息如下：
```shell
2019-12-20 20:10:56.484 [ERRO] 500, "GET http 127.0.0.1:8199 /log/error HTTP/1.1" 0.210, 127.0.0.1, "", "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36"
Stack:
1. OMG
   1).  main.main.func1
        /Users/john/Workspace/Go/GOPATH/src/github.com/gogf/gf/.example/net/ghttp/server/log/log_error.go:10

```
错误信息会打印出对应错误产生的堆栈信息（堆栈信息中不包含框架内部调用信息），以便于错误定位以及开发者分析问题原因。

> `WebServer`产生的任何`panic`错误都将会被自动捕获到错误日志中，因此对于业务端程序来讲，无论是在控制器中、业务封装层、数据模型中，如果产生了错误想要直接退出业务请求处理，直接`panic`即可。特别是对于绝大多数`golang`风格的方法`error`返回值判断，处理将会变得异常的简单。如果是业务程序自身定义的方法`error`甚至可不用返回，可以根据业务场景选择直接`panic`。

## 输出日志到文件

我们可以通过`SetLogPath`方法来设置日志目录，这样我们的日志信息将会保存到文件中，示例如下：

```go
package main

import (
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/net/ghttp"
)

func main() {
    s := g.Server()
    s.BindHandler("/log/path", func(r *ghttp.Request){
        r.Response.Writeln("请到/tmp/gf.log目录查看日志")
    })
    s.SetLogPath("/tmp/gf.log")
    s.SetAccessLogEnabled(true)
    s.SetErrorLogEnabled(true)
    s.SetPort(8199)
    s.Run()
}
```
运行并访问后，日志将会被保存到`/tmp/gf.log`目录中。手动在终端查看目录结构如下：
```shell
$ tree gf.log
/tmp/gf.log
├── 2019-12-20.log
└── access-20191220.log
```
需要注意的是，其中的`2019-12-20.log`是应用的默认日志文件，往往是通过使用`glog`/`g.Log()`方式输出的业务或者模块日志。而`access-*.log`/`error*.log`日志文件记录的是`WebServer`的访问日志及错误日志。



## 自定义日志处理

`gf`支持自定义的日志处理特性，便于开发者灵活控制日志输出，使用`SetLogHandler`来实现日志回调方法注册。

> 也可以使用中间件或者事件回调来实现自定义的日志输出，具体请查看对应【[中间件](net/ghttp/router/middleware.md)】和【[事件回调](net/ghttp/router/hook.md)】章节。

以下是自定义日志处理的简单示例：
```go
package main

import (
    "fmt"
    "net/http"
    "github.com/gogf/gf/frame/g"
    "github.com/gogf/gf/net/ghttp"
)

func main() {
    s := g.Server()
    s.BindHandler("/log/handler", func(r *ghttp.Request){
        r.Response.WriteStatus(http.StatusNotFound, "文件找不到了")
    })
    s.SetAccessLogEnabled(true)
    s.SetErrorLogEnabled(true)
    s.SetLogHandler(func(r *ghttp.Request, error ...interface{}) {
        if len(error) > 0 {
            // 如果是错误日志
            fmt.Println("错误产生了：", error[0])
        }
        // 这里是请求日志
        fmt.Println("请求处理完成，请求地址:", r.URL.String(), "请求结果:", r.Response.Status)
    })
    s.SetPort(8199)
    s.Run()
}
```

运行并请求后，终端输出内容如下：
```shell
请求处理完成，请求地址: /log/handler 请求结果: 404
```