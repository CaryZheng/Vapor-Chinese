# Views
---
Views 从 application 返回 HTML 数据。他们可以由纯 HTML 文档创建或通过渲染器，比如 Mustache 或 Stencil 。


## Views Directory 
---
Views 被存储在 ```Resources/Views``` 中。该目录可通过 application 的 views directory 属性来访问。

##### Swift
```
app.viewsDir
```

## HTML
---
返回 HTML 或 任何非渲染的文档是简单的。只需要传递文档路径到到 ```View(path: String)``` 。该路径是相对于 ```viewsDir``` 的。

##### Swift
```
app.get("/") { request in
    return try View(path: "index.html")
}
```

> Throws
> 
> 如果未能找到该文件，该调用会抛出一个异常 ```View.Error.InvalidPath``` 。你可以在闭包或中间件捕获它。

## Public Resources
--- 
Views 需要的任何资源，比如图片，类型和脚本，都应该存放在 application 根目录中的 ```Public``` 文件夹中

## View Renderer
---
任意符合 ```ViewRenderer``` 的类都能使用已有的 context 来渲染  views 。

##### Swift
```
class MustacheRenderer: RenderDriver {
    ...
}

View.renderers[".mustache"] = MustacheRenderer()
```

这将使得任何以 ```.mustache``` 结尾的文件在返回之前都能使用 mustache 渲染器。

## Available Renderers
---
这些渲染器可以通过 Providers 来添加到你的 application 中。

## Mustache
* [Zewo Mustache](https://github.com/qutheory/vapor-mustache)

## Stencil
* [Qutheory Stencil](https://github.com/qutheory/vapor-stencil)

