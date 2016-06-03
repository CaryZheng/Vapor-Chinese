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

## 命令行
---
除了内嵌在 ```Config/``` 目录下的 json 文件外，我们也可以使用命令行来传递参数到我们的配置。默认情况下，这些值将会被设置在 ```app``` 文件中，但更复杂的选项也是可以设置的。

#### 1. --KEY=VALUE

命令行设置的参数可以通过配置的 app 文件来访问。例如，如下的 CLI 命令：

##### Text
```
--mongo-password=$MONGO_PASSWORD
```

通过如下操作在你的 application 中将可以访问到。

##### Swift
```
let mongoPassword = app.config["app", "mongo-password"].string
```

#### 2. --CONFIG:FILE-NAME.KEY=CUSTOM-VALUE

除了 ```app``` 文件外，你也可以使用更高级的方法。例如，如下的 CLI 命令：

##### Text
```
--config:keys.analytics=124ZH61F
```

通过如下操作在你的 application 中将可以访问到：

##### Swift
```
let analyticsKey = app.config["keys", "analytics"].string
```

## 高级配置
---
默认的 ```app.json``` 不错，但是如何应对更复杂的情景。例如，要是我们想要生产和开发拥有不同的 host 怎么办？这些复杂情景可以通过在 Config/  目录下添加额外的文件夹来解决。如下是用于生产和开发环境的设置。

##### Text
```
WorkingDirectory/
├── Config/
│   ├── app.json
│   ├── production/
│   │   └── app.json
│   ├── development/
│   │   └── app.json
│   └── secrets/
│       └── app.json
```

> 通过使用命令行 ```--env=``` 你可以指定具体环境。自定义环境也是支持的，默认提供 ```production``` 、 ```development``` 和 ```testing``` 。

##### Text
```
vapor run --env=production
```

#### 优先级
配置文件按如下优先级进行访问。

1. CLI (see below)
2. Config/secrets/
3. Config/name-of-environment/
4. Config/

这意味着如果用户调用 ```app.config["app", "host"]``` ，key将首先在 CLI 中搜索，然后 secrets 目录，接着是顶级的默认配置。

> ```secrets/``` 目录应该被添加到 gitignore 。

#### 例子

让我们从如下的 JSON 文件开始。

##### app.json
```
{
  "host": "0.0.0.0"
  "port": 9000
}
```

##### production/app.json
```
{
  "host": "127.0.0.1"
  "port": 80
}
```

##### secrets/app.json
```
{
  "mongo-user": "secret-user",
  "mongo-pass": "secret-pass"
}
```

请注意 ```app.json``` 和 ```production/app.json``` 都声明了同样的 key ， 即 ```host``` 和 ```port``` 。 在 application 中，我们可以这样调用：

##### Swift
```
// will load 0.0.0.0 or 127.0.0.1 based on above config
let host = app.config["app", "host"].string
// will load 9000, or 80 based on above config
let port = app.config["app", "port"].int
```

根据当前环境，将会加载对应的值。 secrets 文件也是按相同方式访问。

##### Swift
```
let user = app.config["app", "mongo-user"].string
let pass = app.config["app", "mongo-pass"].string
```

如果 secrets 文件不存在，这些值将会是 ```nil``` 。