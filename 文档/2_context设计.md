期望：
将路由(router)独立出来，方便之后增强。
设计上下文(Context)，封装 Request 和 Response ，提供对 JSON、HTML 等返回类型的支持。

web服务根据请求request，构造响应response，每次请求我们都要构造完整的响应，需要编写重复且麻烦的代码
```go
obj = map[string]interface{}{
    "name": "geektutu",
    "password": "1234",
}
w.Header().Set("Content-Type", "application/json")
w.WriteHeader(http.StatusOK)
encoder := json.NewEncoder(w)
if err := encoder.Encode(obj); err != nil {
    http.Error(w, err.Error(), 500)
}
```
封装之后我们只要考虑消息的主体内容
```go
c.JSON(http.StatusOK, gee.H{
    "username": c.PostForm("username"),
    "password": c.PostForm("password"),
})
```
## 构造Context结构体及其构造方法
在gee下新建context.go
```go
// 类型别名H，map 键类型为string，值为任意类型
type H map[string]interface{}

type Context struct{
	// 源对象
	Writer http.ResponseWriter
	Req *http.Request

	// 请求路径与方式
	Path string
	Method string

	// 状态码
	StatusCode int
}

func newContext(w http.ResponseWriter,req *http.Request) *Context{
	return &Context{
		Writer: w,
		Req: req,
		Path: req.URL.Path,
		Method: req.Method,
	}
}
```
## 封装响应(json、html等)
```go
func (c *Context)PostForm(key string)string{
	return c.Req.FormValue(key)
}

// 请求路径里面的请求参数
func (c *Context)Query(key string)string{
	return c.Req.URL.Query().Get(key)
}

// 将状态码写入响应
func (c * Context)Status(code int){
	c.StatusCode=code
	c.Writer.WriteHeader(code)
}

func (c * Context)SetHeader(key string,value string){
	c.Writer.Header().Set(key,value)
}

func (c * Context)String(code int,format string,values ...interface{}){
	c.SetHeader("Content-Type", "text/plain")
	c.Status(code)
	c.Writer.Write([]byte(fmt.Sprintf(format,values...)))
}

func (c *Context)JSON(code int,obj interface{}){
	c.SetHeader("Content-Type","application/json")
	c.Status(code)
	encoder:=json.NewEncoder(c.Writer)
	if err:=encoder.Encode(obj);err!=nil{
		http.Error(c.Writer,err.Error(),500)
	}
}

func (c *Context)Data(code int,data []byte){
	c.Status(code)
	c.Writer.Write(data)
}

func (c *Context)HTML(code int,html string){
	c.SetHeader("Content-Type","text/html")
	c.Status(code)
	c.Writer.Write([]byte(html))
}
```

## 提取路由相关的东西
在gee下新建router.go
```go
package gee

import (
	"log"
	"net/http"
)

type router struct{
	handlers map[string]Handlerfunc
}

func newRouter()*router{
	return &router{
		handlers: make(map[string]Handlerfunc),
	}
}

func (r *router)addRoute(method string,pattern string,handler Handlerfunc){
	log.Printf("Route %4s - %s",method,pattern)
	key:=method+"-"+pattern
	r.handlers[key]=handler
}

func (r *router)handle(c *Context){
	key:=c.Method+"-"+c.Path
	if handler,ok:=r.handlers[key];ok{
		handler(c.Writer,c.Req)
	}else{
		c.String(http.StatusNotFound,"404 NOT FOUND: %s\n",c.Path)
	}
}
```
