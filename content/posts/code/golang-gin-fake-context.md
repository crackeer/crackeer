---
title: "Golang Gin创建假的Context"
date: 2021-02-21T08:52:02+08:00
tags: ["代码片段"]
categories: ["IT"]
---

```golang
urlParameter := &url.Values{}
query := map[string]string{
	"relation_id": "1098963",
	"source":      "alliance",
	"platform":    "shell",
	"channel":     "3",
	"house_code":  "101104214655",
	"work_code":   "vdrj5DR7PGW3q8Oe",
	"user_id":     "0",
}
for key, value := range query {
	urlParameter.Set(key, value)
}

ctx.Request, _ = http.NewRequest("GET", "http://localhost:9200/house/card/show.json", strings.NewReader(urlParameter.Encode()))

ctx.Request.Header = http.Header(map[string][]string{
	"User-Agent": {"PostmanRuntime/7.26.8"},
})
```

