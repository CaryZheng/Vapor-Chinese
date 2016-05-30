# 安装CLI

## 安装CLI
---
Vapor命令行接口对通用任务提供了快捷帮助。让我们来安装它。

##### Shell
```
curl -L cli.qutheory.io -o vapor
chmod +x vapor
sudo mv vapor /usr/local/bin
```

> Swift是必需的
> 
> Vapor CLI是Swift脚本文件，因此你需要先安装Swift 3，并在对应PATH使用它。


## 验证
---
通过运行help查询来确保CLI已成功安装。你应该看见可用命令的输出。

##### Shell
```
vapor help
```

## 更快
---
你可以编译这个脚本使得它运行的更快。

##### Shell
```
vapor self compile
```

## 更新
---
CLI可以自我更新。当你遇到任何问题的时候这可能会有帮助的。

##### Sehll
```
vapor self update
```