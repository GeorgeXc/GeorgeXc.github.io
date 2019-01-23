# GO语言orm框架gorm

## 简单示例

```go
package main

import (
	"time"

	"github.com/jinzhu/gorm"
	_ "github.com/jinzhu/gorm/dialects/mysql"
)

type User struct {
	Name     string
	Age      int
	BirthDay time.Time
}

func main() {
	db, err := gorm.Open("mysql", "user:password@/dbname?charset=utf8&parseTime=True&loc=Local")
	if err != nil {
		return
	}
	defer db.Close()
    user := User{}
    var animal = Animal{Age: 99, Name: ""}
}
```

## gorm语句含义对照表

| gorm语句             | 含义                                         | 备注                  | 吐槽  |
|--------------------|--------------------------------------------|---------------------|-----|
| `db.Create(&user)` | 创建插入该值到数据库中                                | 但是 如果有0值字段，则不能把0值写入 |     |
| `db.First(&user)`  | `SELECT * FROM users ORDER BY id LIMIT 1;` | 函数名称不能明确表达意思        |     |

                                                                         |
                                                                          |
| `db.Take(&user)`                                                                            | 读取一条记录                                                                                                             | 你知道读取那一条吗？            |         |                                                                          |
| `db.Last(&user)`                                                                            | 读取最后一条记录 `SELECT * FROM users ORDER BY id DESC LIMIT 1;`                                                           |                       |         |                                                                          |
| `db.Find(&users)`                                                                           | 读取所有记录                                                                                                             | 不是find吗，总给点条件来查找吧     |         |                                                                          |
| `db.Where("name = ?", "jinzhu").First(&user)`                                               | `SELECT * FROM users WHERE name = 'jinzhu' limit 1;`                                                               |                       |         |                                                                          |
| `db.Where("name = ?", "jinzhu").Find(&users)`                                               | `SELECT * FROM users WHERE name = 'jinzhu';`                                                                       | 获取所有匹配的记录             |         | `db.Where("name in (?)", []string{"jinzhu", "jinzhu 2"}).Find(&users)`   |
| `db.Where("name LIKE ?", "%jin%").Find(&users)`                                             |                                                                                                                    | 能加limit,offset 吗      |         |                                                                          |
| `db.Where("name = ? AND age >= ?", "jinzhu", "22").Find(&users)`                            |                                                                                                                    | 能用外面的and来设置多个条件吗      |         |                                                                          |
| `db.Where("updated_at > ?", lastWeek).Find(&users)`                                         |                                                                                                                    |                       |         |                                                                          |
| `db.Where("created_at BETWEEN ? AND ?", lastWeek, today).Find(&users)`                      |                                                                                                                    |                       |         |                                                                          |
| `db.Where(&User{Name: "jinzhu", Age: 20}).First(&user)`                                     | `SELECT * FROM users WHERE name = "jinzhu" AND age = 20 LIMIT 1;`                                                  | 如果age是0，则不能构成查询条件     | 太奇葩了吧   |                                                                          |
| `db.Where(map[string]interface{}{"name": "jinzhu", "age": 20}).Find(&users)`                | `SELECT * FROM users WHERE name = "jinzhu" AND age = 20; `                                                         |                       |         |                                                                          |
| `db.Where([]int64{20, 21, 22}).Find(&users)`                                                | `SELECT * FROM users WHERE id IN (20, 21, 22);`                                                                    |                       |         |                                                                          |
| `db.Where(&User{Name: "jinzhu", Age: 0}).Find(&users)`                                      | `SELECT * FROM users WHERE name = "jinzhu";`                                                                       |                       |         |                                                                          |
| `db.Not("name", "jinzhu").First(&user)`                                                     | `SELECT * FROM users WHERE name <> "jinzhu" LIMIT 1;`                                                              |                       |         |                                                                          |
| `db.Not("name", []string{"jinzhu", "jinzhu 2"}).Find(&users)`                               | `SELECT * FROM users WHERE name NOT IN ("jinzhu", "jinzhu 2");`                                                    |                       |         |                                                                          |
| `db.Not([]int64{1,2,3}).First(&user)`                                                       | `SELECT * FROM users WHERE id NOT IN (1,2,3);`                                                                     |                       |         |                                                                          |
| `db.Not([]int64{}).First(&user)`                                                            | `SELECT * FROM users;`                                                                                             |                       |         |                                                                          |
| `db.Not("name = ?", "jinzhu").First(&user)`                                                 | `SELECT * FROM users WHERE NOT(name = "jinzhu");`                                                                  |                       |         |                                                                          |
| `db.Not(User{Name: "jinzhu"}).First(&user)`                                                 | `SELECT * FROM users WHERE name <> "jinzhu";`                                                                      |                       |         |                                                                          |
| `db.Where("role = ?", "admin").Or("role = ?", "super_admin").Find(&users)`                  | `SELECT * FROM users WHERE role = 'admin' OR role = 'super_admin';`                                                |                       |         |                                                                          |
| `db.Where("name = 'jinzhu'").Or(User{Name: "jinzhu 2"}).Find(&users)`                       | `SELECT * FROM users WHERE name = 'jinzhu' OR name = 'jinzhu 2';`                                                  |                       |         |                                                                          |
| `db.Where("name = 'jinzhu'").Or(map[string]interface{}{"name": "jinzhu 2"}).Find(&users)`   | `SELECT * FROM users WHERE name = 'jinzhu' OR name = 'jinzhu 2';`                                                  |                       |         |                                                                          |
| `db.First(&user, 23)`                                                                       | `SELECT * FROM users WHERE id = 23 LIMIT 1;`                                                                       |                       |         |                                                                          |
| `db.First(&user, "id = ?", "string_primary_key")`                                           | `SELECT * FROM users WHERE id = 'string_primary_key' LIMIT 1;`(Get by primary key if it were a non-integer type)   |                       |         |                                                                          |
| `db.Find(&user, "name = ?", "jinzhu")`                                                      | `ELECT * FROM users WHERE name = "jinzhu";`                                                                        |                       |         |                                                                          |
| `db.Find(&users, "name <> ? AND age > ?", "jinzhu", 20)`                                    | `SELECT * FROM users WHERE name <> "jinzhu" AND age > 20;`                                                         |                       |         |                                                                          |
| `db.Find(&users, User{Age: 20})`                                                            | `SELECT * FROM users WHERE age = 20;`                                                                              |                       |         |                                                                          |
| `db.Find(&users, map[string]interface{}{"age": 20})`                                        | `SELECT * FROM users WHERE age = 20;`                                                                              |                       |         |                                                                          |
| `db.Set("gorm:query_option", "FOR UPDATE").First(&user, 10)`                                | ` SELECT * FROM users WHERE id = 10 FOR UPDATE;`                                                                   |                       |         |                                                                          |
| `db.FirstOrInit(&user, User{Name: "non_existing"})`                                         | 不存在在创建一条                                                                                                           |                       |         |                                                                          |
| `db.Where(User{Name: "Jinzhu"}).FirstOrInit(&user)`                                         |                                                                                                                    |                       |         |                                                                          |
| `db.FirstOrInit(&user, map[string]interface{}{"name": "jinzhu"})`                           |                                                                                                                    |                       |         |                                                                          |
| `db.Select("name, age").Find(&users)`                                                       | `SELECT name, age FROM users;`                                                                                     |                       |         |                                                                          |
| `db.Select([]string{"name", "age"}).Find(&users)`                                           | `SELECT name, age FROM users;`                                                                                     |                       |         |                                                                          |
| `db.Table("users").Select("COALESCE(age,?)", 42).Rows()`                                    | `SELECT COALESCE(age,'42') FROM users;`                                                                            |                       |         |                                                                          |


看了上面这么多的用法，你眼花了没?
上面这么多用法，你记住了几种，知道其中的特别注意事项吗？
我敢肯定你一定记不住。
一个orm，设计的这么难用我也是醉了。