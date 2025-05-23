---
title: "设计模式-系统设计"
date: 2023-03-17T23:00:40+08:00
categories:
- 计算机科学与技术
- 设计模式
tags:
- 设计模式
- 暂停施工
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/design-pattern.svg
---
系统设计是成为成熟程序员的一个标志，先来看看前辈们的设计吧
<!--more-->
## 长连接系统架构设计

## 资金账户系统设计
1. 系统分层：
    - 顶层业务：如金融服务、监管合规、基础支付、业务创新
    - 接口：网关、收银台、快捷支付
    - 底层：
        - 支付业务：交易系统、支付核心、支付业务、提现业务
        - 会员系统：商户管理、会员管理、密码、银行卡
        - 资金系统：账户管理、账务核心、账单管理、对账管理、核算报表
        - 渠道业务：渠道代付、签约管理
    > 各部分都要根据业务需要提供：记账、查询、管理、通知能力
1. 难点：业务对一致性、持久性、高并发等能力要求都很高
1. 技术能力支撑：
    - 高并发：将账户金额划分成balance、freeze、unreach、system四个部分，引入额外字段，提高并发性
    - 稳定性：保证事务的提交和回滚都能正确执行即可
    - 分布式事务：主事务 + 分支事务，事务管理器、事务恢复daemon
    - 异步化记账
> 注：关键思路是将账户资金进行部分冻结，以这种细化的方式，提高系统并发性和稳定性