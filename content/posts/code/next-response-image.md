---
title: "Nextjs返回图片"
date: 2021-02-28T10:47:58+08:00
tags: ["代码片段"]
categories: ["IT"]
---

#### Show me your code

```js

export default async (req, res) => {
    const {
        query: { name },
    } = req

    let fileName = './data/image/global/' + name
    console.log(fileName)
    if (fs.existsSync(fileName)) {
        res.writeHead(200, { 'Content-Type': 'image/png' });
        var stream = fs.createReadStream(fileName);
        var responseData = [];//存储文件流
        if (stream) {//判断状态
            stream.on('data', function (chunk) {
                responseData.push(chunk);
            });
            stream.on('end', function () {
                var finalData = Buffer.concat(responseData);
                res.write(finalData);
                res.end();
            });
        }
        return
    }
    res.statusCode = 404
    res.write("")
}
```

