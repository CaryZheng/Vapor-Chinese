# Asymmetric Cryptography 非对称密码算法

非对称密码算法（也称为公钥密码算法）是使用多个密钥的密码系统 - 通常是“公共”和“私人”密钥。

参阅 Wikipedia 上关于 [public-key cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography)更多内容。

## RSA

一种流行的非对称密码算法是RSA。 RSA有两种关键类型：公共和私人。

RSA可以使用私钥从任何数据创建签名。

```swift
let privateKey: String = ...
let signature = try RSA.SHA512.sign("vapor", key: .private(pem: privateKey))
```

!!! 资料
	只有私钥可以用 _create_ 签名。

可以使用公钥或私钥对这些签名进行相同的数据验证。

```swift
let publicKey: String = ...
try RSA.SHA512.verify(signature, signs: "vapor", key: .public(pem: publicKey)) // true
```

如果RSA验证签名与公共密钥的输入数据匹配，则可以确定生成该签名的人可以访问该密钥的私钥。

### Algorithms

RSA 支持任意Crypto模块里的 [`DigestAlgorithm`](https://api.vapor.codes/crypto/latest/Crypto/Classes/DigestAlgorithm.html).

```swift
let privateKey: String = ...
let signature512 = try RSA.SHA512.sign("vapor", key: .private(pem: privateKey))
let signature256 = try RSA.SHA256.sign("vapor", key: .private(pem: privateKey))
```