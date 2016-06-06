# 安装Swift 3 (Ubuntu)
---
安装Vapor，你需要先安装Swift 3。

## 下载
---
Linux上安装Swift仅需要花费一点的时间，同时支持 14.04 和 15.10。只有下载区域有所不同。


> <font color="#56C0E0"> Swift Version Manager </font>
> 
> 如果你在多个Swift 3项目中工作，你可能需要看看“安装Swift 3 (Swiftenv)”章节。


## 下载（14.04 - Trusty Tahr）
---
##### Shell
```
# Download Swift
curl -O https://swift.org/builds/development/ubuntu1404/swift-DEVELOPMENT-SNAPSHOT-2016-05-31-a/swift-DEVELOPMENT-SNAPSHOT-2016-05-31-a-ubuntu14.04.tar.gz

# Untar
tar -zxf swift-DEVELOPMENT-SNAPSHOT-2016-05-31-a-ubuntu14.04.tar.gz

# Rename
mv swift-DEVELOPMENT-SNAPSHOT-2016-05-31-a-ubuntu14.04 swift
```

## 下载（15.10 - Wily Werewolf）
---
##### Shell
```
# Download
wget https://swift.org/builds/development/ubuntu1510/swift-DEVELOPMENT-SNAPSHOT-2016-05-09-a/swift-DEVELOPMENT-SNAPSHOT-2016-05-09-a-ubuntu15.10.tar.gz

# Untar
tar -zxf swift-DEVELOPMENT-SNAPSHOT-2016-05-09-a-ubuntu15.10.tar.gz

# Rename
mv swift-DEVELOPMENT-SNAPSHOT-2016-05-09-a-ubuntu15.10 swift
```

## 安装依赖
---

取决于使用Ubuntu的偏好，你可能需要安装一些附加工具用于编译器。我们将以可靠方式提示错误信息来安装你需要的一切。

##### Shell
```
sudo apt-get install clang libicu-dev binutils git
```

## 安装
---
安装Swift有两种方式。要么添加到你的PATH，要么迁移到你的bin目录下。

##### Shell
```
# Add to path
echo "export PATH=/path/to/swift/usr/bin:\"\${PATH}\"" >> ~/.bashrc

# Reload path
source ~/.bashrc
```

> <font color="#F2AE43"> 绝对路径 </font>
> 
> 确保使用Swift安装后的绝对路径替换 ```/path/to/swift/```


## 验证
---
检查你的安装是否有效。

##### Shell
```
swift --version

# Apple Swift version 3.0-dev (LLVM b010debd0e, Clang 3e4d01d89b, Swift 7182c58cb2)
```