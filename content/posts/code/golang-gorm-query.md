---
title: "Golang Gorm通用的query查询函数"
date: 2021-03-03T22:22:02+08:00
tags: ["日记"]
categories: ["生活记录"]
---

#### golang、gorm、query

利用反射获取表名，以及分表长度和分区键

```golang
package storage

import (
	"context"
	"errors"
	"fmt"
	"reflect"
	"time"

	pkgUtil "git.lianjia.com/vrlab_server/gopkg/util"
	"gorm.io/gorm"
)

// Option ...
type Option struct {
	Context context.Context

	//查询相关
	TableDestType *TableDestType

	Field string
	Order string

	//缓存相关，业务层使用
	Cache    time.Duration // 缓存时间
	NilCache bool          //nil也缓存

	Timeout time.Duration // Context 超时时间
}

// TableDestType ...
type TableDestType struct {
	Name      string
	RealName  string
	SplitSize int64
	SplitKey  string
	Kind      reflect.Kind
}

// Query ...
func Query(db *gorm.DB, query map[string]interface{}, dest interface{}, opt *Option) error {

	if db == nil {
		return errors.New("database connection nil")
	}

	if opt.TableDestType == nil {
		var err error
		opt.TableDestType, err = ParseDestTableType(dest)
		if err != nil {
			return err
		}
	}

	if opt.TableDestType.SplitSize > 1 && len(opt.TableDestType.SplitKey) > 0 {
		splitKeyVal := pkgUtil.GetInt64ValFromMap(query, opt.TableDestType.SplitKey)
		opt.TableDestType.RealName = fmt.Sprintf("%s_%d", opt.TableDestType.Name, splitKeyVal%opt.TableDestType.SplitSize)
	}

	db = db.Table(opt.TableDestType.RealName)

	// 添加where条件
	simpleMap, sliceMap := parseQuery(query)
	if len(simpleMap) > 0 {
		db = db.Where(simpleMap)
	}
	if len(sliceMap) > 0 {
		for k, v := range sliceMap {
			db = db.Where(fmt.Sprintf("%s in ?", k), v)
		}
	}

	//设置查询字段、orderBy字段、context
	if opt != nil {
		if len(opt.Field) > 0 {
			db = db.Select(opt.Field)
		}

		if len(opt.Order) > 0 {
			db = db.Order(opt.Order)
		}

		if opt.Context != nil {
			db = db.WithContext(opt.Context)
		}
	}

	//查询
	if opt.TableDestType.Kind == reflect.Struct {
		if len(opt.Order) > 0 {
			return db.Take(dest).Error
		}
		return db.Last(dest).Error
	}
	return db.Find(dest).Error
}

// ParseDestTableType ...
func ParseDestTableType(dest interface{}) (*TableDestType, error) {

	destType := reflect.TypeOf(dest)
	if destType.Elem().Kind() != reflect.Slice && destType.Elem().Kind() != reflect.Struct {
		return nil, errors.New("dest must be addressable")
	}

	retData := &TableDestType{
		Kind: destType.Elem().Kind(),
	}

	var obj reflect.Value
	if retData.Kind == reflect.Struct {
		obj = reflect.New(destType.Elem())
	} else {
		obj = reflect.New(destType.Elem().Elem())
	}

	//获取table_name
	f1 := obj.MethodByName("TableName")
	if f1.Kind() == reflect.Invalid {
		return nil, errors.New("dest must have method `TableName`")
	}
	tableName := f1.Call(nil)
	if len(tableName) < 1 {
		return nil, errors.New("get table name fail")
	}
	retData.Name = tableName[0].String()

	//获取分表size
	f2 := obj.MethodByName("TableSplit")
	if f2.Kind() != reflect.Invalid {
		res := f2.Call(nil)

		if len(res) > 0 {
			retData.SplitSize = res[0].Int()
			retData.SplitKey = res[1].String()
		}
	}

	if retData.SplitSize < 2 {
		retData.RealName = retData.Name
	}

	return retData, nil
}

func parseQuery(query map[string]interface{}) (map[string]interface{}, map[string]interface{}) {

	sliceMap := map[string]interface{}{}
	simpleMap := map[string]interface{}{}
	for k, v := range query {
		destType := reflect.TypeOf(v)
		if destType.Kind() == reflect.Slice {
			sliceMap[k] = v
		} else {
			simpleMap[k] = v
		}
	}

	return simpleMap, sliceMap
}

```

