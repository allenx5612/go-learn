## 使用Group方法注册路由组，以达到产品模块分组开发的目地
##### 如：/user/login, /user/register, /user/delete

```golang
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
)

func main() {
	engine := gin.Default()

	// 通过gruop方法注册路由组, 如/user/login, /user/register组
	routerGroup := engine.Group("/user")
	routerGroup.POST("/register", registerHandler)
	routerGroup.POST("/login", loginHandler)
	// 同样使用:定义变量
	routerGroup.DELETE("/:id", deleteHandler)

	engine.Run()
}

func registerHandler(context *gin.Context) {
	fullPath := "用户注册： " + context.FullPath()
	fmt.Println(fullPath)
	context.Writer.WriteString(fullPath)
}

func loginHandler(context *gin.Context) {
	fullPath := "用户登录： " + context.FullPath()
	fmt.Println(fullPath)
	context.Writer.WriteString(fullPath)
}

func deleteHandler(context *gin.Context) {
	fullPath := "用户删除： " + context.FullPath()
	userID := context.Param("id")
	fmt.Println(fullPath, userID)
	context.Writer.WriteString(fullPath + " " + userID)
}
```
