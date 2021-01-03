## Gin返回数据、JSON以及HTML页面的方法
1. 后端方法实现
```golang
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"net/http"
)

// context.JSON()演示结构体
type Response struct {
	Code    int         `json:"code"`
	Message string      `json:"message"`
	Data    interface{} `json:"data"`
}

func main() {
	engine := gin.Default()

	// 通过context.Writer方法

	// 通过context.Writer.Write()返回数据
	engine.GET("/hellobyte", func(context *gin.Context) {
		fullPath := "请求路径: " + context.FullPath()
		fmt.Println(fullPath)
		context.Writer.Write([]byte(fullPath))
	})

	// 通过context.Writer.WriteString()返回数据
	engine.GET("/hellostring", func(context *gin.Context) {
		fullPath := "请求路径: " + context.FullPath()
		fmt.Println(fullPath)
		context.Writer.WriteString(fullPath)
	})

	// 返回JSON

	// 通过context.JSON()并使用map返回JSON数据
	engine.GET("/hellojson", func(context *gin.Context) {
		fullPath := "请求路径: " + context.FullPath()
		fmt.Println(fullPath)

		context.JSON(200, map[string]interface{}{
			"code":    1,
			"message": "OK",
			"data":    fullPath,
		})
	})

	// 通过context.JSON()并使用struct返回JSON数据
	engine.GET("/hellostruct", func(context *gin.Context) {
		fullPath := "请求路径: " + context.FullPath()
		fmt.Println(fullPath)

		resp := Response{Code: 1, Message: "OK", Data: fullPath}
		context.JSON(200, &resp)
	})

	//返回HTML页面

	// 使用该方法设置HTML文件根目录
	engine.LoadHTMLGlob("./forms/*.html")

	// Static方法设置前端相对路径与本地路径的映射
	engine.Static("/images", "./forms/images")
	engine.Static("/static", "./forms/static")

	engine.GET("/hellohtml", func(context *gin.Context) {
		fullPath := "请求路径: " + context.FullPath()
		fmt.Println(fullPath)

		// 设置跟目录后，name不用跟上路径, obj可通过模板语言返回变量到前端，也可为nil
		// 模板格式见前端
		context.HTML(http.StatusOK, "index.html", gin.H{
			"fullPath": fullPath,
			"title":    "hello",
		})
	})

	engine.Run()
}
```
2. 前端HTML与后端HTML方法的交互
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{{.title}}</title>
    <link rel="stylesheet" href="static/index.css">
    <script type="text/javascript" src="static/index.js"></script>
</head>

<body>
<h1>hello</h1>
<!--后端返回数据通过模板变量显示方法-->
<h2>{{.fullPath}}</h2>
<!--静态资源文件需通过后端Static方法映射-->
<img src="images/Backward.png" alt="backward">
</body>

</html>
```
