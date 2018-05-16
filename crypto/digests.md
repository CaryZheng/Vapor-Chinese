# Message Digests 消息摘要

加密哈希函数 (又称为报文摘要算法) convert data of arbitrary size to a fixed-size digest. These are most often used for generating checksums or identifiers for large data blobs.

Read more about [Cryptographic hash functions](https://en.wikipedia.org/wiki/Cryptographic_hash_function) on Wikipedia.

## Hash 哈希

利用全局便利变量去创建使用常用算法的哈希。

```swift
import Crypto

let digest = try SHA1.hash("hello")
print(digest.hexEncodedString()) // aaf4c61ddcc5e8a2dabede0f3b482cd9aea9434d
```

参阅 Crypto 模块中的 [global variables](https://api.vapor.codes/crypto/latest/Crypto/Global%20Variables.html#/Digests) 里的一系列可用的哈希算法。

### Streaming 

你可以手动创建一个 [`Digest`](https://api.vapor.codes/crypto/latest/Crypto/Classes/Digest.html) 然后用它的实例方法创建一个或多个数据块的哈希。

```swift
var sha256 = try Digest(algorithm: .sha256)
try sha256.reset()
try sha256.update(data: "hello")
try sha256.update(data: "world")
let digest = try sha256.finish()
print(digest) /// Data
```

## BCrypt

BCrypt是一种流行的哈希算法，具有可配置的复杂性并自动加salting。

### Hash

使用 `hash(_:cost:salt:)` 去创建BCrypt哈希。

```swift
let digest = try BCrypt.hash("vapor", cost: 4)
print(digest) /// data
```

随着`cost` 值的增加，哈希加密和验证会花费更多的时间。

### Verify

使用`verify(_:created:)` 方法去验证一个由给定明文输入的BCrypt哈希加密。

```swift
let hash = try BCrypt.hash("vapor", cost: 4)
try BCrypt.verify("vapor", created: hash) // true
try BCrypt.verify("foo", created: hash) // false
```

## HMAC

HMAC是一种用于创建 _keyed_ 哈希算法。如果使用不同的密钥，HMAC将为相同的输入生成不同的哈希值。

```swift
let digest = try HMAC.SHA1.authenticate("vapor", key: "secret") 
print(digest.hexEncodedString()) // digest
```

参阅 [`HMAC`](https://api.vapor.codes/crypto/latest/Crypto/Classes/HMAC.html) 类的一系列可用的哈希算法。

### Streaming

HMAC哈希也可以流式处理。它的API和 [hash streaming](#streaming)一样。