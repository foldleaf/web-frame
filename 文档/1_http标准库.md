
go有内置的net/http标准库，封装了http网络编程的接口
```go
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
func http.HandleFunc(pattern string, handler func(http.ResponseWriter, *http.Request))

func http.ListenAndServe(addr string, handler http.Handler) error
```
