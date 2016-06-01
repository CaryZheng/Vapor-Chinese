# Config
<font color="#999999">应用配置设置</font>

---
使用配置来给 app 添加自定义行为。

## 快速开始
---
在应用的工作目录下，应该有个 ```Config/``` 子目录，在这里可以找到配置文件。首先我们将添加 ```app.json``` ， 目录看起来像这样：

##### Text
```
./
├── Config/
│   ├── app.json
```
有一些 key 是 Vapor 默认使用的。现在让我们自定义我们应用的 ```host``` 和 ```port``` 。现在 ```app.json``` 文件看起来像这样：

##### JSON
```
{
  "host": "0.0.0.0",
  "port": 8080
}
```

现在当应用加载的时候，它将自动运行在 8080 端口。

### 自定义 KEYS
让我们添加一个自定义 key 到 ```app.json``` 文件中。

##### JSON
```
{
  "host": "0.0.0.0",
  "port": 8080,
  "custom-key": "custom value"
}
```

可以在你的应用配置文件中使用如下方式来访问：

##### Swift
```
let customValue = app.config["app", "custom-key"].string
```

就这样，按需利用 keys 来使你的应用配置更简单。

## 配置语法
---
你可以使用如下语法来访问配置目录： ```app.config[<#file-name#>, <#path#>, <#to#>, <#file#>]``` 。例如，除了我们之前提到的 ```app.json``` 文件外，也可以有 ```keys.json``` ，看起来像这样：

##### JSON
```
{
  "test-names": [
    "joe",
    "jane",
    "sara"
   ]
  "mongo": {
    "url" : "www.customMongoUrl.com"
  }
}
```

要访问该文件，请确保下标第一个参数是 ```keys``` 。去获取列表中的第一个 name 示例如下。

##### Swift
```
let name = app.config["keys", "test-names", 0].string
```

或者 mongo url：

##### Swift
```
let mongoUrl = app.config["keys", "mongo", "url"].string
```

# 未完待续...