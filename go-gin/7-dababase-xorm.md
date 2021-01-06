## xorm的使用
1. 使用xorm连接数据库，数据库设置，以及查询的方法
```golang
package main

import (
	"fmt"
	_ "github.com/go-sql-driver/mysql"
	"github.com/go-xorm/xorm"
	"xorm.io/core"
)

func main() {
	db, err := xorm.NewEngine("mysql", "root:password@(localhost:3306)/test?charset=utf8")
	if err != nil {
		panic(err.Error())
	}
	defer db.Clone()

	// 数据库引擎设置
	db.ShowSQL(true)                     // 设置显示sql语句
	db.Logger().SetLevel(core.LOG_DEBUG) //设置日志级别
	db.SetMaxOpenConns(10)               // 设置最大连接数
	db.SetMaxIdleConns(2)                // 设置最大空闲连接数，默认为2

	// 查询数据
	session := db.Table("app_key")
	count, err := session.Count()
	if err != nil {
		panic(err.Error())
	}
	fmt.Println(count)

	result, err := db.Query("select * from app_key")
	if err != nil {
		panic(err.Error())
	}
	for _, row := range result {
		for key, value := range row {
			fmt.Println(key, string(value))
		}
	}
}
```
2. 结构体与数据库的映射
```golang
package main

import (
	"fmt"
	_ "github.com/go-sql-driver/mysql"
	"github.com/go-xorm/xorm"
	"xorm.io/core"
)

type UserTable struct {
	UserId   int64  `xorm:"pk autoincr"`
	UserName string `xorm:"varchar(32)"`
	UserAge  int64  `xorm:"default 1"`
	UserSex  int64  `xorm:"default 0"`
}

type StudentTable struct {
	Id          int64  `xorm:"pk autoincr"`
	StudentName string `xorm:"varchar(24)"`
	StudentAge  int    `xorm:"int default 0"`
	StudentSex  int    `xorm:"index"`
}

type PersonTable struct {
	ID         int64     `xorm:"pk autoincr"`
	PersonName string    `xorm:"varchar(24)"`
	PersonAge  int       `xorm:"int default 0"`
	PersonSex  int       `xorm:"notnull"`
	City       CityTable `xorm:"-"` // 使用-忽略映射字段
}

type CityTable struct {
	CityName      string
	CityLongitude float32
	CityLatitude  float32
}

func main() {
	db, err := xorm.NewEngine("mysql", "root:password@(localhost:3306)/test?charset=utf8")
	if err != nil {
		panic(err.Error())
	}
	defer db.Clone()

	// 设置结构体映射关系
	db.SetMapper(core.SnakeMapper{}) // 驼峰映射
	//db.SetMapper(core.SameMapper{})  // 相同映射
	//db.SetMapper(core.GonicMapper{}) // 与驼峰映射类似，但在ID等名称上会智能识别为id，而不是i_d
	db.Sync2(new(UserTable))
	db.Sync2(new(StudentTable))
	db.Sync2(new(PersonTable))

	// 判断表是否为空
	personEmpty, err := db.IsTableEmpty(new(PersonTable))
	if err != nil {
		panic(err.Error())
	}
	if personEmpty {
		fmt.Println("person table is empty")
	} else {
		fmt.Println("person table is not empty")
	}

    // 判断表是否存在
	studentExists, err := db.IsTableExist(new(StudentTable))
	if err != nil {
		panic(err.Error())
	}
	if studentExists {
		fmt.Println("student table exists")
	} else {
		fmt.Println("student table not exists")
	}
}
```
**tag数据定义关键字：**
| tag关键字             | 对应数据库名称                                      |
| ---                   | ---                                                 |
| pk                    | primary key                                         |
| autoincr              | 自增                                                |
| null或notnull         | 是否为null                                          |
| unique                | 是否时唯一                                          |
| index                 | 是否是索引                                          |
| extends               | 应用于匿名成员结构体或非匿名成员结构体              |
| -                     | 字段不映射                                          |
| ->                    | 只写入                                              |
| <-                    | 只读取                                              |
| created               | insert时自动赋值当前时间                            |
| updated               | insert或update时自动赋值当前时间                    |
| deleted               | delete时设置为当前时间，并且当前记录不删除          |
| version               | insert时默认为1,每次更新自动加1                     |
| default 0或default(0) | 设置默认值，紧跟的内容如果是varchar等需要加上单引号 |
| json                  | 表示内容将先转成JSON格式                            |

**数据类型的映射关系:**
| Go数据类型                                           | xorm中的类型 |
| ---                                                  | ---          |
| Implement Conversion                                 | Text         |
| int, int8, int16, int32, uint, uint8, uint16, uint32 | Int          |
| int64, uint64                                        | Bigint       |
| float32                                              | Float        |
| float64                                              | Double       |
| complex64, complex128                                | Varchar(64)  |
| []uint8                                              | Blob         |
| array, slice, map except []uint8                     | Text         |
| bool                                                 | Bool         |
| string                                               | Varchar(255) |
| time.Time                                            | DateTime     |
| cascade struct                                       | Bigint       |
| struct                                               | Text         |
| Others                                               | Text         |
3. xorm增删改查以及事物的使用方法
```golang
package main

import (
	"fmt"
	_ "github.com/go-sql-driver/mysql"
	"github.com/go-xorm/xorm"
	"xorm.io/core"
)

type PersonTable struct {
	Id         int64  `xorm:"pk autoincr"`
	PersonName string `xorm:"varchar(24)"`
	PersonAge  int    `xorm:"int default 0"`
	PersonSex  int    `xorm:"notnull"`
}

func main() {
	db, err := xorm.NewEngine("mysql", "root:password@(localhost:3306)/test?charset=utf8")
	if err != nil {
		panic(err.Error())
	}
	defer db.Clone()

	// 数据库引擎设置
	db.Logger().SetLevel(core.LOG_DEBUG) //设置日志级别

	// 设置结构体映射关系
	db.ShowSQL(true)
	db.SetMapper(core.SnakeMapper{}) // 驼峰映射
	db.Sync2(new(PersonTable))

	personArray := []PersonTable{
		{
			PersonName: "root",
			PersonAge:  20,
			PersonSex:  1,
		},
		{
			PersonName: "davie",
			PersonAge:  23,
			PersonSex:  2,
		},
		{
			PersonName: "tom",
			PersonAge:  25,
			PersonSex:  1,
		},
		{
			PersonName: "jonny",
			PersonAge:  26,
			PersonSex:  2,
		},
		{
			PersonName: "pony",
			PersonAge:  26,
			PersonSex:  2,
		},
		{
			PersonName: "tony",
			PersonAge:  30,
			PersonSex:  1,
		},
		{
			PersonName: "jack",
			PersonAge:  28,
			PersonSex:  1,
		},
		{
			PersonName: "mali",
			PersonAge:  28,
			PersonSex:  1,
		},
		{
			PersonName: "ruby",
			PersonAge:  28,
			PersonSex:  1,
		},
		{
			PersonName: "rubin",
			PersonAge:  25,
			PersonSex:  1,
		},
	}

	// 事物的用法1
	session := db.NewSession()
	defer session.Clone()
	if err = session.Begin(); err != nil {
		panic(err.Error())
	}
	for _, ptable := range personArray {
		if _, err = session.Insert(&ptable); err != nil {
			session.Rollback()
			panic(err.Error())
		}
	}
	if err = session.Commit(); err != nil {
		panic(err.Error())
	}

	// 事物的用法2
	//_, err = db.Transaction(func(session *xorm.Session) (interface{}, error) {
	//	for _, ptable := range personArray {
	//		if _, err = session.Insert(&ptable); err != nil {
	//			session.Rollback()
	//			return nil, err
	//		}
	//	}
	//	return nil, nil
	//})
	//if err != nil {
	//	panic(err.Error())
	//}

	// 条件查询
	var person PersonTable
	result, err := db.Id(1).Get(&person)
	fmt.Println(result, err)
	fmt.Println(person.PersonName)

	// where多条件查询，get只能获取第一条记录，所有记录需使用find
	var personArray1 []PersonTable
	db.Where("person_age = ? and person_sex = ?", 26, 2).Find(&personArray1)
	fmt.Println(personArray1)

	// 使用And方法
	var personArray2 []PersonTable
	db.Where("person_age=?", 26).And("person_sex = ?", 2).Find(&personArray2)
	fmt.Println(personArray2)

	// 使用Or方法
	var personArray3 []PersonTable
	db.Where("person_age=?", 26).Or("person_sex = ?", 2).Find(&personArray3)
	fmt.Println(personArray3)

	// 使用原生sql语句查询
	var personNative []PersonTable
	err = db.SQL("select * from person_table where person_name like '%t'").Find(&personNative)
	if err != nil {
		panic(err.Error())
	}
	fmt.Println(personNative)

	// 查询结果排序
	var personOrderBy []PersonTable
	err = db.OrderBy("person_age desc").Find(&personOrderBy)
	if err != nil {
		panic(err.Error())
	}
	fmt.Println(personOrderBy)

	// 查询特定字段
	var personCols []PersonTable
	err = db.Cols("person_name", "person_age").Find(&personCols)
	if err != nil {
		panic(err.Error())
	}
	fmt.Println(personCols)

	// 增
	personInsert := PersonTable{
		PersonName: "hello",
		PersonAge:  18,
		PersonSex:  1,
	}
	rowNum, err := db.Insert(&personInsert)
	if err != nil {
		panic(err.Error())
	}
	fmt.Println(rowNum, personInsert)

	// 删
	rowNum, err = db.Delete(&personInsert)
	if err != nil {
		panic(err.Error())
	}
	fmt.Println(rowNum)

	// 改
	rowNum, err = db.ID(7).Update(&personInsert)
	if err != nil {
		panic(err.Error())
	}
	fmt.Println(rowNum)

	// 统计
	count, err := db.Count(new(PersonTable))
	if err != nil {
		panic(err.Error())
	}
	fmt.Println(count)
}
```
