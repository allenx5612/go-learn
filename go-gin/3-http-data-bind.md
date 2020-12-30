## Golang使用结构体接收数据
```golang
package main

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"log"
)

// GET示例结构体
type Student struct {
	Name    string `form:"name"`
	Classes string `form:"classes"`
}

// POST接收form-data示例结构体
type Register struct {
	UserName string `form:"username" binding:"required"`
	Phone    string `form:"phone" binding:"required"`
	Password string `form:"password" binding:"required"`
}

// POST接收JSON示例结构体, 注意数据类型要与前端JSON字段类型对齐
type Person struct {
	Name string `form:"name"`
	Sex  string `form:"sex"`
	Age  int    `form:"age"`
}

func main() {
	engine := gin.Default()

	// http://localhost:8080/hello?name=dave&classes=software-engineering
	engine.GET("/hello", func(context *gin.Context) {
		fmt.Println(context.FullPath())

		var student Student
		if err := context.ShouldBindQuery(&student); err != nil {
			log.Fatal(err.Error())
			return
		}

		fmt.Println(student.Name, student.Classes)
		context.Writer.Write([]byte("hello " + student.Name))
	})

	// POST接收form-data
	engine.POST("/register", func(context *gin.Context) {
		fmt.Println(context.FullPath())

		var register Register
		if err := context.ShouldBind(&register); err != nil {
			log.Fatal(err.Error())
			return
		}
		fmt.Println(register.UserName, register.Phone, register.Password)

		context.Writer.Write([]byte(register.UserName + " register"))
	})

	// POST接收JSON数据
	engine.POST("/addstudent", func(context *gin.Context) {
		fmt.Println(context.FullPath())

		var person Person
		if err := context.BindJSON(&person); err != nil {
			log.Fatal(err.Error())
		}
		fmt.Println(person.Name, person.Sex, person.Age)

		context.Writer.Write([]byte("add student " + person.Name))
	})

	engine.Run()
}
```
