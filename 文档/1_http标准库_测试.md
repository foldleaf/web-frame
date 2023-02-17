
## http.HandleFunc
go有内置的net/http标准库，封装了http网络编程的接口
```go
// test1.go
package main

import (
	"fmt"
	"log"
	"net/http"
)

func main() {
	// 访问路径 “/”，访问 indexHandler 函数
	http.HandleFunc("/", indexHandler)

	// 访问路径 “/hello”，调用 helloHandler 函数
	http.HandleFunc("/hello", helloHandler)

	// http.ListenAndServe 在指定端口监听
	log.Fatal(http.ListenAndServe(":9999", nil))
}

// 打印请求路径 r.URL.Path
func indexHandler(w http.ResponseWriter, req *http.Request) {
	fmt.Fprintf(w, "URL.Path=%q\n", req.URL)
}

// 打印请求头 r.URL.Header
func helloHandler(w http.ResponseWriter, req *http.Request) {
	for k, v := range req.Header {
		fmt.Fprintf(w, "Header[%q]=%q\n", k, v)
	}
}
```
那么其中的http.HandleFunc与http.ListenAndServe
```go
// 第一个参数是路由，第二个参数是一个函数
func http.HandleFunc(pattern string, handler func(http.ResponseWriter, *http.Request))
// 第一个参数是地址,第二个参数则代表处理所有的HTTP请求的实例，nil 代表使用标准库中的实例处理
func http.ListenAndServe(addr string, handler http.Handler) error
```

字段 Header http.Header：Header 包含由服务器接收或由客户端发送的请求头字段。
如果服务器收到带有header行的请求,如
```html
Host: example.com
accept-encoding: gzip, deflate
Accept-Language: en-us
fOO: Bar
foo: two
```
那么会转换成
```go
Header = map[string][]string{
    "Accept-Encoding": {"gzip, deflate"},
    "Accept-Language": {"en-us"},
    "Foo": {"Bar", "two"},
}
```
使用curl命令行工具测试
```bash
# 测试输入
> curl http://localhost:9999/
# 测试输出
URL.Path="/"

# 测试输入
> curl http://localhost:9999/hello  
# 测试输出
Header["User-Agent"]=["curl/7.87.0"]
Header["Accept"]=["*/*"]
```
## http.Handler接口
```go
再来看 http.ListenAndServe
package http

type Handler interface {
    ServeHTTP(w ResponseWriter, r *Request)
}

func ListenAndServe(address string, h Handler) error
```
它的第二个参数是一个Handler类型的实例。Handler是一个接口，只要是实现了方法 ServeHTTP(w ResponseWriter, r *Request) 就是实现了 Handler 接口

```go
// test2.go
package main

import (
	"fmt"
	"log"
	"net/http"
)

type Engine struct{

}
// 属于 Engine 结构体的方法，实现了 Handler 接口的 ServeHTTP 方法 --> 实现了 Handler 接口
func (engine *Engine) ServeHTTP(w http.ResponseWriter,req *http.Request){
	switch req.URL.Path{
		case "/":
			fmt.Fprintf(w, "URL.Path = %q\n", req.URL.Path)
		case "/hello":
			for k,v:=range req.Header{
				fmt.Fprintf(w, "Header[%q] = %q\n", k, v)
			}
		default:
			fmt.Fprintf(w,"404 NOT FOUND:%s\n",req.URL)
	}
}


func main(){
	// Engine 结构体实现了Handler接口，其实例可以作为参数传入函数 http.ListenAndServe
	engine:=new(Engine)
	log.Fatal(http.ListenAndServe(":9999",engine))
}
```
同样可以使用curl命令行工具测试

## 总结
我们自定义了一个结构体，通过实现handler接口，可以将 http 请求转向我们自定义的规则进行逻辑处理

