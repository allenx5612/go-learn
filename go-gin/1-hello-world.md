## Hello world例子
```golang
package main

import (
	"github.com/gin-gonic/gin"
	"fmt"
	"log"
)

func main() {
	engine := gin.Default()
	engine.GET("/hello", func(context *gin.Context) {
		fmt.Println("请求路径： ", context.FullPath())
		context.Writer.Write([]byte("Hello gin\n"))
	})

	if err := engine.Run(":8090"); err != nil {
		log.Fatal(err.Error())
	}
}

```