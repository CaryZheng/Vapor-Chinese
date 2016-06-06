# Command Line Arguments
---
启动的时候你可以通过命令行参数来覆写特定的 Vapor 设置。这允许你在不需要重编译 application 的情况下动态更改设置。

## 工作目录
---
默认情况下， Vapor 使用当前工作目录作为工程的根目录。如果你使用 ```vapor run``` 或者其它来自工程根目录的方法的话都会起作用。然而，如果你运行可执行文件，但所在目录并不是工程根目录的话， application 将不知道去哪找到 ```Public``` 或 ```Resources``` 文件夹。

为了避免上述情况，传递工程的根目录作为 ```--workDir``` 。例如：

##### Text
```
/home/username/project/.build/debug/App --workDir=/home/username/project/
```

## Host and Port
---
根据你的服务器配置，你可能需要更改默认的 host 或绑定的 port 。

只需要分别传递 ```--ip``` 或 ```port``` 即可。例如：

##### Text
```
vapor run --ip=192.168.0.1 --port=9000
```