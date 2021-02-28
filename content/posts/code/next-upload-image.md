---
title: "NextJS上传图片"
date: 2021-02-27T18:04:26+08:00
tags: ["代码片段"]
categories: ["IT"]
---

#### 代码

```js

import { IncomingForm } from 'formidable'
// you might want to use regular 'fs' and not a promise one
import { promises as fs } from 'fs'

// first we need to disable the default body parser
export const config = {
    api: {
        bodyParser: false,
    }
}

export default async (req, res) => {
    // parse form with a Promise wrapper
    const data = await new Promise((resolve, reject) => {
        const form = new IncomingForm()

        form.parse(req, (err, fields, files) => {
            if (err) return reject(err)
            resolve({ fields, files })
        })
    })
    console.log(data.files.file)
}
```

