# 安装

Ubuntu上安装Vapor只需要花费几分钟时间。

## 支持

Vapor支持的Ubuntu版本与Swift一致。

Version | Codename
----|----
16.10 | Yakkety Yak
16.04 | Xenial Xerus
14.04 | Trusty Tahr

# APT Repo

添加Vapor APT repo，以便于访问Vapor所有的Ubuntu包。

## 快速开始

使用如下脚本可以方便地添加Vapor的APT repo。

```
eval "$(curl -sL https://apt.vapor.sh)"
```

> 提示：该命令需要```curl```，可以通过```sudo apt-get install curl```来安装。

## Dockerfile

想通过Dockerfile来配置Ubuntu，可以通过如下命令来添加APT repo：

```
RUN /bin/bash -c "$(wget -qO- https://apt.vapor.sh)"
```

## 手动

或者手动添加repo。

```
wget -q https://repo.vapor.codes/apt/keyring.gpg -O- | sudo apt-key add -
echo "deb https://repo.vapor.codes/apt $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/vapor.list
sudo apt-get update
```

# 安装Vapor

目前已经添加了Vapor的APT repo，你可以安装必要的依赖了。

```
sudo apt-get install swift vapor
```

## 验证安装

通过如下命令来检查是否安装成功。

### Swift

```
swift --version
```

将会看到类似如下的输出：

```
Apple Swift version 4.0.2 (swiftlang-900.0.69.2 clang-900.0.38)
Target: x86_64-apple-macosx10.9
```

Vapor需要Swift 4。

### Vapor Toolbox

```
vapor --help
```

你将会看到一长串可用命令集。

## 完成

Vapor安装好后，就可以创建第一个应用了，可参考 [Getting Started → Hello, world](../getting_started/hello_world.md)

## Swift.org

如果查阅 [Swift.org](https://swift.org/)的指南中的[using downloads](https://swift.org/download/#using-downloads)。