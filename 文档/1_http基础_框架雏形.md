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

