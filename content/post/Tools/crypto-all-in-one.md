---
title: "信息安全专栏"
date: 2023-10-26T16:17:36+08:00
categories:
- 计算机科学与技术
- 信息安全
tags:
- 信息安全
- 施工中
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/security.jpg
---
本文记录所有的信息安全中的加解密知识，库，实用工具。也包括鉴权、访问控制、抗抵赖相关知识。
<!--more-->

## OpenSSL
- 注意点
    1. OpenSSL的风格是，内部结构**不导出**，因此对于```ECDSA_SIG```、```BIGNUM```等，不能直接通过成员访问操作符```->```来获取成员。正确的方式是使用相对应的API。例如
        ```c
            // 对ASN.1的DER编码进行解码
        	ECDSA_SIG* result = d2i_ECDSA_SIG(&sig, (const unsigned char**)&derData, derlen);
            // BIGNUM也是用BN_new获取
            const BIGNUM* pr = BN_new();
            const BIGNUM* ps = BN_new();
            // 对result->r和result->s的获取
            ECDSA_SIG_get0(result, &pr, &ps);
        ```
- Windows下的编译：
    - 步骤：
        1. 安装Perl
        2. ```mkdir your-build ; cd your-build```
        3. ```perl Configure VC-WIN64A no-asm --prefix=D:\path\to\install```，生成makefile等，更多[配置说明](https://wiki.openssl.org/index.php/Compilation_and_Installation)
        4. 调用Visual Studio的Native Tools Command Prompt，执行```nmake```
        5. ```nmake install```
    - 参考：[win10 x64 visual studio 2019 编译 OpenSSL-1.1.1](https://blog.csdn.net/qiuxue110/article/details/115560240)

## 鉴权
1. 基础术语
    1. Cookie
    2. Token
    3. 跨域
2. 单点登录：在一个多应用系统中，提供一处登录处处登录，一处登出处处登出的效果。参考[单点登录解决方案](https://cloud.tencent.com/developer/article/1636664)。
    1. Cookie
    2. SSO
    3. OAuth
    4. JWT（JSON Web Token）
3. OAuth2.0