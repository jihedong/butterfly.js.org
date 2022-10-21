---
title: consul官方文档译文
date: 2022-10-21 09:51:04
categories:
- 注册中心
- consul
tags:
---
# Consul 是什么？
HashiCorp Consul 是一个服务网络解决方案，使团队能够管理服务之间的跨本地和多云环境以及运行时的安全网络连接。Consul 提供服务发现、服务网格、流量管理和网络基础设施设备自动更新。您可以在 Consul 单机或集群环境中使用这些特性。

# Consul 是怎么工作的？
Consul提供了一个 *control plane*，使您能够注册、查询和保护跨网络部署的服务。*control plane*  是网络基础设施的一部分，它维护一个中央注册中心来跟踪服务及其各自的IP地址。它是一个运行在节点集群(如物理服务器、云实例、虚拟机或容器)上的分布式系统。

Consul 通过代理与 *data plane* 交互。*data plane* 是处理数据请求的网络基础设施的一部分。详情请参阅 [Consul Architecture](https://developer.hashicorp.com/consul/docs/architecture)。

{% asset_img assets.png Consul工作架构图 %}

Consul 工作流程包括以下阶段:
- **服务注册**：将服务添加到 Consul 目录中服务就可以自动发现彼此，而不需要开发人员修改源代码、部署额外的负载平衡器或硬编码IP地址。它是所有服务及其地址的运行时来源。团队可以使用 CLI 或 API 手动[定义和注册服务](https://developer.hashicorp.com/consul/docs/discovery/services)，或者您可以使用 [service sync](https://developer.hashicorp.com/consul/docs/k8s/service-sync) 在 Kubernetes 中自动化这个过程。服务还可以包括健康检查，以便 Consul 可以监控不健康的服务。
- **服务查询**：Consul 基于身份的 DNS 让您可以在 Consul 目录中找到健康的服务。向 Consul 注册的服务提供健康信息、接入点和其他数据可以帮助您控制网络中的数据流。根据您定义的基于身份的策略，您的服务只能通过其本地代理访问其他服务。
- **服务安全**：服务在被定义为上游之后，Consul 确保服务到服务的通信被认证、授权和加密。Consul service mesh 通过 MTL 保护微服务架构，并可以基于服务身份允许或限制访问，并不需要考虑计算机环境和运行时的差异。