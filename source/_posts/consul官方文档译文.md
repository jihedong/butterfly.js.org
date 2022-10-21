---
title: consul官方文档译文
date: 2022-10-21 09:51:04
categories:
- 注册中心
- consul
tags:
---
# 1、Consul 是什么？
HashiCorp Consul 是一个服务网络解决方案，使团队能够管理服务之间的跨本地和多云环境以及运行时的安全网络连接。Consul 提供服务发现、服务网格、流量管理和网络基础设施设备自动更新。您可以在 Consul 单机或集群环境中使用这些特性。

# 2、Consul 是怎么工作的？
Consul提供了一个 *control plane*，使您能够注册、查询和保护跨网络部署的服务。*control plane*  是网络基础设施的一部分，它维护一个中央注册中心来跟踪服务及其各自的IP地址。它是一个运行在节点集群(如物理服务器、云实例、虚拟机或容器)上的分布式系统。

Consul 通过代理与 *data plane* 交互。*data plane* 是处理数据请求的网络基础设施的一部分。详情请参阅 [Consul Architecture](https://developer.hashicorp.com/consul/docs/architecture)。

{% asset_img assets.png Consul工作架构图 %}