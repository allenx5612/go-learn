## GIN处理HTTP请求
1. GIN利用Handle处理HTTP请求
```golang
package main

import (
	"github.com/gin-gonic/gin"
	"fmt"
)

func main() {
	engine := gin.Default()

    // get请求示例 http://localhost:8080/hello?name=dave
	engine.Handle("GET", "/hello", func(context *gin.Context) {
		path := context.FullPath()
		fmt.Println(path)

		name := context.DefaultQuery("name", "hello")
		fmt.Println(name)

		context.Writer.Write([]byte("Hello, " + name))
	})

	engine.Handle("POST", "/login", func(context *gin.Context) {
		fmt.Println(context.FullPath())

		username := context.PostForm("username")
		pwsword := context.PostForm("password")
		fmt.Println(username)
		fmt.Println(pwsword)

		context.Writer.Write([]byte(username + "登录"))
	})

	engine.Run()
}

```

2. 利用本身的Get，Post等方法处理HTTP请求
```golang
package main

import (
	"github.com/gin-gonic/gin"
	"fmt"
)

func main() {
	engine := gin.Default()

	engine.GET("/hello", func(context *gin.Context) {
		fmt.Println(context.FullPath())

		// Query没有DefaultQuery第二参数的默认值
		name := context.Query("name")
		fmt.Println(name)

		context.Writer.Write([]byte("hello " + name))
	})

	engine.POST("/login", func(context *gin.Context) {
		fmt.Println(context.FullPath())

		username, exists := context.GetPostForm("username")
		if exists {
			fmt.Println(username)
		}

		pwswords, exists := context.GetPostForm("password")
		if exists {
			fmt.Println(pwswords)
		}

		context.Writer.Write([]byte("hello " + username))
	})
    
    	// 使用:定义变量值
	engine.DELETE("/user/:id", func(context *gin.Context) {
		// Param方法获取变量id
		userID := context.Param("id")
		fmt.Println(userID)
		context.Writer.Write([]byte("delete user " + userID))
	})

	engine.Run()
}
```

