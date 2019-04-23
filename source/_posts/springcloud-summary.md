---
title: SpringCloud 概述
date: 2019-01-11 20:41:32
categories: SpringCloud
tags: [java, SpringCloud]
---

# SpringCloud 概述

## 微服务

微服务是一个小的、松耦合的分布式服务。

### 1、微服务的基础问题

> 服务粒度

* 服务职责单一
* 服务无状态话
* 操作表不能太多（3-5个）
* 避免简单的 CRUD服务

> 通信协议

> 接口设计

* RESTful 风格接口
* 内容使用 JSON
* HTTP 状态码表示结果

> 服务的配置管理 -- Config

	统一配置服务器，避免配置与服务硬绑定。

> 服务之间的事件处理 -- Stream

	使用事件解耦微服务，最小化服务之间的硬编码依赖。

### 2、微服务路由模式

负责处理客服端的服务请求，使其到达特定实例。

> 服务路由（网关） -- Netflix Zuul

	为所有服务提供单个入口点，可整合安全策略、路由规则等。
	
> 服务发现 -- Netflix Eureka

	微服务注册中心、管理服务。

	
### 3、客服端弹性模式

避免单个服务影响整个系统，提高服务稳定性。

> 客服端负载均衡 -- Neflix Ribbon

	负载均衡，避免单个服务过载

> 断路器模式 -- Netflix Hystrix

	阻止客户继续调用出故障/有性能问题的服务

> 后备模式 -- Netflix Hystrix

	服务调用失败后，提供后备方案

> 舱壁模式 -- Netflix Hystrix

	避免个别微服务故障影响整个服务。

### 4、微服务安全

> 验证 -- Security/OAuth2

	身份验证

> 授权 -- Security/OAuth2

	权限验证

> 凭据管理和传播  -- Security/OAuth2 JWT

	验证的模式 -JWT

### 5、微服务日志记录与跟踪模式

> 日志关联 -- Sleuth

	关联请求在所有微服务中的调用链。

> 日志聚合 -- Sleuth、Papertrail

	将所有微服务的日志聚合到一起

> 微服务跟踪 -- Sleuth/Zipkin

	跟踪微服务的事物流程，以及其性能。

### 6、微服务构建和部署

> 构建和部署管道

	可重复的构建和部署过程。

> 基础设施即代码

	

> 不可变服务器

	部署之后永远不会改变

> 凤凰服务器（Phoenix server）

	服务长期一致性。


![](http://qnyunyun.yunsoho.cn/20180626135647743.jpeg?imageMogr2/thumbnail/!100p)
















































































































































