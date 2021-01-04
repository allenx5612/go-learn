## Gin使用中间件的方法，以及context.Next()的使用
```golang
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {
	engine := gin.Default()

	// Use方法注册中间件, Use方法中间件会影响全局的方法
	engine.Use(RequestInfos())
	engine.GET("/query", func(context *gin.Context) {
		context.JSON(http.StatusOK, map[string]interface{}{
			"code": 1,
			"msg":  context.FullPath(),
		})
	})

	// 单独为某个方法添加中间件的方法, 无需在Use方法中注册
	engine.GET("/query", RequestInfos(), func(context *gin.Context) {
		context.JSON(http.StatusOK, map[string]interface{}{
			"code": 1,
			"msg":  context.FullPath(),
		})
	})

	engine.Run()
}

// 利用中间件每次打印输出请求path和method，中间件函数需返回gin.HandlerFunc类型
func RequestInfos() gin.HandlerFunc {
	return func(context *gin.Context) {
		path := context.FullPath()
		method := context.Request.Method
		fmt.Println("请求Path: ", path)
		fmt.Println("请求Method: ", method)
		// context.Next()将中间件中断，转而执行具体业务逻辑，再返回中间件继续执行
		context.Next()
		// 利用context.Next()函数即可获得处理后的状态码
		fmt.Println("状态码: ", context.Writer.Status())
	}
}
```
