
[TOC]


`GF`框架的`Web Server`配置管理非常方便，支持多种配置方式以及若干配置方法。

# 配置对象

配置对象定义：
https://godoc.org/github.com/gogf/gf/net/ghttp#ServerConfig



## 配置管理方法
方法列表： https://godoc.org/github.com/gogf/gf/net/ghttp#Server

简要说明：
1. 可以通过`SetConfig`及`SetConfigWithMap`来设置。
1. 也可以使用`Server`对象的`Set*/Enable*`方法进行特定配置的设置。
1. 主要注意的是，配置项在`Server`执行`Start`之后便不能再修改，以便产生并发安全问题。

## `SetConfigWithMap`方法

我们可以使用`SetConfigWithMap`方法通过`Key-Value`键值对来设置/修改`Server`的特定配置，其余的配置使用默认配置即可。其中`Key`的名称即是`ServerConfig`这个`struct`中的属性名称，并且不区分大小写，单词间也支持使用`-`/`_`/`空格`符号连接，具体可参考【[gconv.Struct转换规则](util/gconv/struct.md)】章节。

简单示例：
```go
s := g.Server()
s.SetConfigWithMap(g.Map{
    "Address":    ":80",
    "ServerRoot": "/var/www/MyServerRoot",
})
s.Run()
```
其中`ServerRoot`的键名也可以使用`serverRoot`, `server-root`, `server_root`, `server root`，其他配置属性以此类推。


一个比较完整的示例：
```go
s := g.Server()
s.SetConfigWithMap(g.Map{
    "Address":          ":80",
    "ServerRoot":       "/var/www/Server",
    "IndexFiles":       g.Slice{"index.html", "main.html"},
    "AccessLogEnabled": true,
    "ErrorLogEnabled":  true,
    "PProfEnabled":     true,
    "LogPath":          "/var/log/ServerLog",
    "SessionIdName":    "MySessionId",
    "SessionPath":      "/tmp/MySessionStoragePath",
    "SessionMaxAge":    24 * time.Hour,
    "DumpRouterMap":    false,
})
s.Run()
```

# 配置文件

从`GF v1.10`版本开始，`Server`对象也支持通过配置文件进行便捷的配置。支持的配置选项以及配置说明请查看接口文档说明，文档中有详细说明，以下章节不会对配置选项作介绍。

当使用`g.Server(单例名称)`获取`Server`单例对象时，将会自动通过默认的配置管理对象获取对应的`Server`配置。默认情况下会读取`server.单例名称`配置项，当该配置项不存在时，将会读取`server`配置项。

## 示例1，默认配置项
```toml
[server]
    Address    = ":80"
    ServerRoot = "/var/www/Server"
```
随后可以使用`g.Server()`获取默认的单例对象时自动获取并设置该配置。

## 示例2，多个配置项
多个`Server`的配置示例：
```toml
[server]
    Address    = ":80"
    ServerRoot = "/var/www/Server"
    [server.server1]
        Address    = ":8080"
        ServerRoot = "/var/www/Server1"
    [server.server2]
        Address    = ":8088"
        ServerRoot = "/var/www/Server2"
```
我们可以通过单例对象名称获取对应配置的`Server`单例对象：
```go
// 对应 server.server1 配置项
s1 := g.Server("server1")
// 对应 server.server2 配置项
s2 := g.Server("server2")
// 对应默认配置项 server
s3 := g.Server("none")
// 对应默认配置项 server
s4 := g.Server()
```

## 示例3，较完整示例
比如上一个章节的示例，对应的配置文件如下：
```toml
[server]
    Address          = ":8199"
    ServerRoot       = "/var/www/Server"
    IndexFiles       = ["index.html", "main.html"]
    AccessLogEnabled = true
    ErrorLogEnabled  = true
    PProfEnabled     = true
    LogPath          = "/var/log/ServerLog"
    SessionIdName    = "MySessionId"
    SessionPath      = "/tmp/MySessionStoragePath"
    SessionMaxAge    = "24h"
    DumpRouterMap    = false
```
同理，配置属性项的名称也不区分大小写，单词间也支持使用`-`/`_`符号连接。也就是说以下配置文件效果和上面的配置文件一致：
```toml
[server]
    address          = ":8199"
    serverRoot       = "/var/www/Server"
    indexFiles       = ["index.html", "main.html"]
    accessLogEnabled = true
    errorLogEnabled  = true
    pprofEnabled     = true
    log-path         = "/var/log/ServerLog"
    session_Id_Name  = "MySessionId"
    Session-path     = "/tmp/MySessionStoragePath"
    session_MaxAge   = "24h"
    DumpRouterMap    = false
```




