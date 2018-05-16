# Cipher Algorithms

密码允许您使用密钥加密明文数据，产生密文。此密文可以稍后使用相同的密钥通过相同的密码进行解密。

参阅 Wikipedia 里的 [ciphers](https://en.wikipedia.org/wiki/Cipher)。

## Encrypt 加密

使用全局便捷变量以通用算法对数据进行加密。

```swift
let ciphertext = try AES128.encrypt("vapor", key: "secret")
print(ciphertext) /// Data
```

## Decrypt 解密

Decryption 和 [encryption](#encrypt) 非常类似. 以下代码片段展示了如何解密我们前面例子中的密文。

```swift
let plaintext = try AES128.decrypt(ciphertext, key: "secret")
print(plaintext) /// "vapor"
```

参阅Crypto 模块里的 [global variables](https://api.vapor.codes/crypto/latest/Crypto/Global%20Variables.html#/Ciphers) 的一系列可用的cipher算法。

## Streaming

加密和解密都可以在允许数据分块的流式下工作。这对于在加密大量数据时控制内存使用情况很有用。

```swift
let key: Data // 16-bytes
let aes128 = Cipher(algorithm: .aes128ecb)
try aes128.reset(key: key, mode: .encrypt)
var buffer = Data()
try aes128.update(data: "hello", into: &buffer)
try aes128.update(data: "world", into: &buffer)
try aes128.finish(into: &buffer)
print(buffer) // Completed ciphertext
```