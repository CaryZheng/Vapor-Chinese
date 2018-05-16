# Random 随机

`Random` 模块处理随机数据的生成，包括随机数的产生。

## Data Generator 数据生成器

[`DataGenerator`]() 类能产生所有随机数据生成器。

### Implementations

- [`OSRandom`](https://api.vapor.codes/crypto/latest/Random/Classes/OSRandom.html): 使用平台特定的方法提供随机数据生成器。


- [`URandom`](https://api.vapor.codes/crypto/latest/Random/Classes/URandom.html) 基于 `/dev/urandom` 文件提供随机数据生成器。


- `Crypto` 模块里的  [`CryptoRandom`](https://api.vapor.codes/crypto/latest/Crypto/Classes/CryptoRandom.html) 使用OpenSSL提供密码安全的随机数据。


```swift
let random: DataGenerator ...
let data = try random.generateData(bytes: 8)
```

### Generate

`DataGenerator`能够使用`generate（_ :)`方法生成随机原始类型。

```swift
let int = try OSRandom().generate(Int.self)
print(int) // Int
```