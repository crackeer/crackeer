---
title: "Nextjs Config"
date: 2021-02-28T11:00:22+08:00
tags: ["代码片段"]
categories: ["生活记录"]
---

#### nextjs配置项解析

https://www.shangmayuan.com/a/c0ae307abeaf47b5ba112a83.html

```json
const withCss = require('@zeit/next-css')

// 配置说明
const configs = {
    // 编译文件的输出目录
    distDir: 'build',
    // 是否给每一个路由生成Etag
    // Etag是用来作缓存验证的，若是路由执行的时候，新的Etag是相同的，那么就会复用当前内容，而无需从新渲染
    // 默认状况下，nextJS是会对每一个路由生成Etag的。可是若是咱们部署的时候，ngx已经作了Etag的配置，那么就能够关闭nextJS的Etag，节省性能
    generateEtags: true,
    // （不经常使用）页面内容缓存配置，只针对开发环境
    onDemandEntries: {
        // 内容在内存中缓存的时长（ms）
        maxInactiveAge: 25 * 1000,
        // 最多同时缓存多少个页面
        pagesBufferLength: 2,
    },
    // 在pages目录下那种后缀的文件会被认为是页面
    pageExtensions: ['jsx', 'js'],
    // （不经常使用）配置buildId，通常用于同一个项目部署多个节点的时候用到
    generateBuildId: async () => {
        if (process.env.YOUR_BUILD_ID) {
        return process.env.YOUR_BUILD_ID
        }

        // 返回null，使用nextJS默认的unique id
        return null
    },
    // （重要配置）手动修改webpack config
    webpack(config, options) {
        return config
    },
    // （重要配置）修改webpackDevMiddleware配置
    webpackDevMiddleware: config => {
        return config
    },
    // （重要配置）能够在页面上经过 procsess.env.customKey 获取 value。跟webpack.DefinePlugin实现的一致
    env: {
        customKey: 'value',
    },
    // 下面两个要经过 'next/config' 来读取
    // 只有在服务端渲染时才会获取的配置
    serverRuntimeConfig: {
        mySecret: 'secret',
        secondSecret: process.env.SECOND_SECRET,
    },
    // 在服务端渲染和客户端渲染均可获取的配置
    publicRuntimeConfig: {
        staticFolder: '/static',
    },
    // 上面这两个配置在组件里使用方式以下：
    // import getCofnig from 'next/config'
    // const { serverRuntimeConfig,publicRuntimeConfig } = getCofnig()
    // console.log( serverRuntimeConfig,publicRuntimeConfig )
}

if (typeof require !== 'undefined') {
    require.extensions['.css'] = file => { }
}

// 虽然next-css看起来是一个处理样式的插件，实则它是接收的一个对象，能够把传入的其余非css相关的webpack配置一并处理。
// 建议不要直接写一个新的webpack配置，由于next-css里面的webpack的配置是很是全面的，若是被覆盖了，可能会致使报错。
module.exports = withCss({
    distDir: 'build'                                                // 这里配置了以后才会生效
})

```

