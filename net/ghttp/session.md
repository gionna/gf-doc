
[TOC]


# 方法列表

与`Session`操作相关的方法：
[godoc.org/github.com/johng-cn/gf/g/net/ghttp#Session](https://godoc.org/github.com/johng-cn/gf/g/net/ghttp#Session)
```go
func (s *Session) BatchSet(m map[string]interface{})
func (s *Session) Contains(k string) bool
func (s *Session) Data() map[string]interface{}
func (s *Session) Get(k string) interface{}
func (s *Session) GetBool(k string) bool
func (s *Session) GetFloat32(k string) float32
func (s *Session) GetFloat64(k string) float64
func (s *Session) GetInt(k string) int
func (s *Session) GetString(k string) string
func (s *Session) GetUint(k string) uint
func (s *Session) Id() string
func (s *Session) Remove(k string)
func (s *Session) Set(k string, v interface{})
func (s *Session) UpdateExpire()

func (s *Server) GetSessionIdName() string
func (s *Server) GetSessionMaxAge() int
func (s *Server) SetSessionIdName(name string)
func (s *Server) SetSessionMaxAge(maxage int)
```
任何时候都可以通过```*ghttp.Request```获取`Session`对象，因为`Cookie`和`Session`都是和请求会话相关，因此都属于`Request`的成员对象，并对外公开。gf框架的`Session`是存放在内存中的，因此处理效率非常高，默认过期时间是`600秒`。

此外，需要说明的是，`Session`的操作是支持`并发安全`的，这也是框架在对`Session`的设计上不采用直接以map的形式操作数据的原因。

在任何时候，我们都可以通过`ghttp.Request`对象来修改和获取`Session`的全局相关属性。

# 使用示例

```go
package main

import (
    "gitee.com/johng/gf/g"
    "gitee.com/johng/gf/g/net/ghttp"
)

func main() {
    s := g.Server()
    s.BindHandler("/session", func(r *ghttp.Request) {
        id := r.Session.GetInt("id")
        r.Session.Set("id", id + 1)
        r.Response.Write("id:", id)
    })
    s.SetPort(8199)
    s.Run()
}
```
启动`main.go`，访问```http://127.0.0.1:8199/session```，刷新几次页面，可以看到页面输出的id值在不断递增。


由于对于Web开发者来讲，`Session`都已经是非常熟悉的组件了，相关API也非常简单，这里便不再赘述。

# Session与Redis整合

`gf`框架提供了`SESSION`模块及`gredis`模块，但是考虑到模块的低耦合性，因此默认情况下并没有做直接的整合处理，有需要的开发者可以按照以下思路便可方便地将`session`与`redis`做整合：
1. 阅读【[事件回调/中间件](net/ghttp/service/hook.md)】章节，及【[Cookie](net/ghttp/cookie.md)】章节；
1. 我们将会用到`Cookie`对象的`SessionId`方法，用于获取客户端提交的`session id`，依靠这个id从`redis`获取/设置数据；
1. 注册`ghttp.HOOK_BEFORE_SERVE`回调函数，在接口服务开始，根据获取到的`session id`去`redis`中查询用户`session`数据，随后通过`SESSION.Sets`方法将从redis中获取的数据设置到`SESSION`中；
1. 注册`ghttp.HOOK_AFTER_SERVE`回调函数，在接口服务结束后，通过`SESSION.Data`方法获取内存中的`session`数据，并将该数据保存到`redis`中；

