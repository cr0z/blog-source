+++
date = "2017-09-07T15:55:24+08:00"
draft = false
title = "golang在MySQL中使用JSON对象"
tags = ["mysql","golang"]

+++

向数据库中插入JSON对象，需要结构体实现Valuer(database/sql/driver包)接口。   
从数据库中读取JSON对象，需要结构体实现Scanner(database/sql包)接口。   
比如发布一条微博或者动态附带一些图片之类的媒体文件,可以直接用JSON对象存入数据库：   
```go
package main

import (
	_ "github.com/go-sql-driver/mysql"
	"database/sql"
	"database/sql/driver"
	"encoding/json"
	"log"
	"time"
)

type Media struct {
	Path string  
	Type int
}

type MediaList []*Media

var _ sql.Scanner = &MediaList{}
var _ driver.Valuer = MediaList{}

//继承Scanner（Scan接受的是指针类型）
func (m *MediaList) Scan(src interface{}) error {
	if src == nil {
		return nil
	}
	b, _ := src.([]byte)
	return json.Unmarshal(b, m)
}

//继承Valuer（INSERT时，Valuer不接受指针类型）
func (m MediaList) Value() (driver.Value, error) {
	return json.Marshal(m)
}

type Weibo struct {
	ID        int
	Author    string
	Content   string
	Media     MediaList
	CreatedAt time.Time
}

func main() {
	db, err := sql.Open("mysql", "yourname:yourpass@tcp(127.0.0.1:3306)/test?parseTime=true")
	if err != nil {
		log.Fatal(err)
	}
	defer db.Close()

	//插入
	weibo := &Weibo{
		Author:  "joe",
		Content: "nice too meet you!",
		Media: MediaList{
			{"http://www.google.com", 1},
			{"http://cn.bing.com", 1},
		},
	}
	_, err = db.Exec("INSERT INTO test_table (author,content,media) VALUES (?,?,?)",
		weibo.Author, weibo.Content, weibo.Media) //这里会调用Valuer(不接受指针类型)
	if err != nil {
		log.Fatal(err)
	} else {
		log.Println("insert success.")
	}

	//查询
	weibo = &Weibo{}
	rows, err := db.Query("SELECT * FROM test_table WHERE id = ? LIMIT 1", 1)
	if err != nil {
		log.Fatal(err)
	}
	for rows.Next() {
		err := rows.Scan(&weibo.ID, &weibo.Author, &weibo.Content, &weibo.Media, &weibo.CreatedAt) //这里会调用Scanner
		if err != nil {
			log.Println(err)
		}
	}
	log.Println(weibo)
}
```
另外，经测试，如使用[Gorm](https://github.com/jinzhu/gorm)框架，实现Scanner和Valuer接口后可直接使用，但如果接口实现不正确，Gorm不会报错，插入/查询的JSON对象为空。   
   
PS：因为需要实现(implemen) interface{}，所以对于数组类型也需要包装成对象后再实现接口,
如：数据库存了 ```[]*YourStruct{}```的JSON数据，需要封装为 ```type Wrapper []*YourStruct```。   