---
title: GORM 自定义数据类型的处理
description: GORM 自定义数据类型
draft: false
date: 2018-05-17
tags:
  - golang
  - orm
  - gorm
---

如果有自定的数据类型，关键是实现 sql 的序列和反序列化方法，即: `driver.Valuer` `sql.Scanner` 接口。

例如有如下数据结构

```go
type StringArray []string

type RecArticle struct {
	Id     uint64 `gorm:"column:id" json:"Id,string"`
	Tags   StringArray  `gorm:"column:tags" json:"tags"`
	Cates  StringArray  `gorm:"column:cates" json:"cates"`
}
```

实现上面的接口

```go
func (t StringArray) Value() (driver.Value, error) {
	b, _ := json.Marshal(t)
	return string(b), nil
}

func (t *StringArray) Scan(src interface{}) error {
	if src != nil {
		var v []string
		json.Unmarshal(src.([]byte), &v)
		*t = v
	}
	return nil
}
```

如果对时间戳，json 都需要特定的处理

add callbacks

```go
// ... init gorm
DB, err = gorm.Open("mysql", "DSN")
// ... handle error

// add callbacks refer: http://gorm.io/docs/write_plugins.html
DB.Callback().Create().After("gorm:update_time_stamp").Register("mark_timestamp_normal", markTimestampNormal)
DB.Callback().Update().After("gorm:update_time_stamp").Register("mark_timestamp_normal", markTimestampNormal)
// ....

func markTimestampNormal(scope *gorm.Scope) {
	if createdAtField, ok := scope.FieldByName("CreatedAt"); ok {
		createdAtField.IsNormal = true
	}

	if updatedAtField, ok := scope.FieldByName("UpdatedAt"); ok {
		updatedAtField.IsNormal = true
	}
}
```

alias

```go
// alias
type Timestamp time.Time

// UnmarshalParam echo api @see https://echo.labstack.com/guide/request
func (t *Timestamp) UnmarshalParam(src string) error {
	if src != "" {
		m, err := strconv.ParseInt(src, 10, 64)
		if err != nil {
			return err
		}

		ts := time.Unix(0, m*int64(time.Millisecond))
		*t = Timestamp(ts)
	}
	return nil
}

// MarshalJSON echo api json response
func (t *Timestamp) MarshalJSON() ([]byte, error) {
	if t != nil {
		ts := time.Time(*t)
		return []byte(fmt.Sprintf(`"%d"`, ts.UnixNano()/int64(time.Millisecond))), nil
	}
	return nil, nil
}

// for sql log, print readable format
func (t Timestamp) String() string {
	ts := time.Time(t)
	return ts.Format(time.RFC3339)
}

// insert into database conversion
func (t Timestamp) Value() (driver.Value, error) {
	return time.Time(t), nil
}

// read from database conversion
func (t *Timestamp) Scan(src interface{}) error {
	if val, ok := src.(time.Time); ok {
		*t = Timestamp(val)
	}
	return nil
}
```

model

```go
type App struct {
	Id        uint32     `json:"id,string"`
	AppKey    string     `json:"appKey" validate:"required"`
	AppSecret string     `json:"appSecret" validate:"required"`
	AppName   string     `json:"appName" validate:"required"`
	CreatedAt Timestamp  `json:"createdAt"`
	UpdatedAt *Timestamp `json:"updatedAt"`
}
```

Refer:

- http://gorm.io/docs/write_plugins.html
- https://github.com/jinzhu/gorm/issues/1236#issuecomment-389633747
- https://echo.labstack.com/guide/request
