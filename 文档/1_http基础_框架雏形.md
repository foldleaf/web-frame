## 文件结构
```bash
|__gee/
    |__gee.go
    |__go.mod
|__main.go
|__go.mod
```
参考gin框架，我们期望通过New()创建一个Handler的实例，通过该实例的请求方法访问路由
```go
func main() {
	r := gee.New()
	r.GET("/", func(w http.ResponseWriter, req *http.Request) {
		fmt.Fprintf(w, "URL.Path = %q\n", req.URL.Path)
	})

	r.GET("/hello", func(w http.ResponseWriter, req *http.Request) {
		for k, v := range req.Header {
			fmt.Fprintf(w, "Header[%q] = %q\n", k, v)
		}
	})

	r.Run(":9999")
}
```
## 框架雏形
在(根目录/)gee/gee.go中
```go
package gee

import (
	"fmt"
	"net/http"
)

// 路由映射
type Handlerfunc func(w http.ResponseWriter, req *http.Request)

// 路由引擎结构体，字段 router 为一个 map，key为string类型（即 http 访问路径），vlalue为路由映射的函数
type Engine struct {
	router map[string]Handlerfunc
}

// 创建路由引擎
func New() *Engine {
	return &Engine{
		router: make(map[string]Handlerfunc),
	}

}

// 添加路由，属于 Engine 类型的方法
// method：“GET” “POST” "PUT" "DELETE" 等请求方法
// pattern: 路由匹配模式
// handler: 路由映射的函数
// 用map的 k/v 建立路由请求与函数映射关系
func (engine *Engine) addRoute(method string, pattern string, handler Handlerfunc) {
	key := method + "-" + pattern
	engine.router[key] = handler
}

// 在 map 中加入预设的路由请求 “GET” “POST” 等
func (engine *Engine) GET(pattern string, handler Handlerfunc) {
	engine.addRoute("GET", pattern, handler)
}
func (engine *Engine) POST(pattern string, handler Handlerfunc) {
	engine.addRoute("POST", pattern, handler)
}

// 实现 Handler 接口的 ServeHTTP 方法
// 在Engine 的map里查找请求的方法路由(method+"-"+pattern),如果存在则返回映射的函数，不存在则404
func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	key := req.Method + "-" + req.URL.Path
	if handler, ok := engine.router[key]; ok {
		handler(w, req)
	} else {
		fmt.Fprintf(w, "404 NOT FOUND:%s\n", req.URL)
	}
}

func (engine *Engine) Run(addr string) (err error) {
	return http.ListenAndServe(addr, engine)
}

```
## 测试框架
在（根目录/）main.go中
```go
package main

import (
	"fmt"
	"frame/gee"
	"net/http"
)

func main(){
	r:= gee.New()
	r.GET("/", func(w http.ResponseWriter, req *http.Request) {
		fmt.Fprintf(w, "URL.Path = %q\n", req.URL.Path)
	})

	r.GET("/hello", func(w http.ResponseWriter, req *http.Request) {
		for k, v := range req.Header {
			fmt.Fprintf(w, "Header[%q] = %q\n", k, v)
		}
	})

	r.Run(":9999")
}
```
使用curl测试

