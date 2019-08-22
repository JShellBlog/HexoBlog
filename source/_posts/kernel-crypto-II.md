---
title: kernel_crypto_II
date: 2019-08-06 20:04:59
tags: cryptographic
categories: drivers
---

### 1. cipher 基础
在Kernel crytographic 的cipher 还有如下几类：
- ablkcipher_tfm(asynchronous multi-block cipher transform)  
- blkcipher_tfm(synchronous multi-block cipher transform)   
- cipher_tfm(Single block cipher transform)

在kernel中cipher 都为skcipher(Symmetric Key cipher, 对称性加密算法)
<!--more-->

后面我们会着重介绍ablkcipher 与blkcipher 这种multi-block 类型的加密算法。

### 2. cipher data structure


### 3. cipher flow

__register & unregister__
```c
int crypto_register_alg(struct crypto_alg *alg);
int crypto_register_algs(struct crypto_alg *algs, int count);

int crypto_unregister_alg(struct crypto_alg *alg);
int crypto_unregister_algs(struct crypto_alg *algs, int count);
```

### 参看资料

