---
title: "Golang实现一个简单的httpServer"
date: 2021-02-21T09:00:49+08:00
tags: ["日记"]
categories: ["生活记录"]
---

### gin

```golang
package main

import (
	"net/http"

	"flag"

	"github.com/gin-gonic/gin"
)

var port *string

func main() {

	port = flag.String("port", "9400", "http port")
	flag.Parse()

	router := gin.Default()
	staticWorkDir := "./dist/"
	router.StaticFile("/", staticWorkDir+"index.html")
	router.StaticFS("/css", http.Dir(staticWorkDir+"css"))
	router.StaticFS("/js", http.Dir(staticWorkDir+"js"))
	router.StaticFS("/img", http.Dir(staticWorkDir+"img"))
	router.StaticFS("/fonts", http.Dir(staticWorkDir+"fonts"))
	router.StaticFS("/static", http.Dir(staticWorkDir+"static"))
	router.Run(":" + *port)

}
```

