---
title: 阿里云 k8s 节约成本记录
tags: 
  - k8s
  - aliyun
date: 2023-01-11
---

# 背景

公司部分阿里云的项目逐步迁入阿里云的 K8S，由于业务场景需求，迁入版本为 ASK，ASK 是构建在 ECI 之上的，所以所有的工作负载，pod 实际上是对应 ECI 的一个容器，如没有明确指定配置或设定 `resource` 默认为 `2cpu,4G` 版本，并且按需付费。

我们有 3 套环境，测试、 灰度和生产，开发环境则部署在公司办公室环境的本地虚拟服务器中。

# 节省方案

## 测试环境

- 无状态负载配置 `Cron HPA` 定时关闭（设置副本数为 `0`）（比如每天凌晨）
- 优化 cpu & 内存配置，尽量使用抢占式实例，突发性能实例能能节省很多，超乎想象
- 尽量使用内网负载均衡器，如果没有外部访问需求，应该创建 `ClusterIP` 的 service
- 实用 `LCU` 计费的负载均衡器和 ingress

## 生产环境

- 优化 cpu & 内存配置，避免浪费，更多副本可能比更高配的机器省钱，更能避免单点故障
- 使用 `LCU` 计费的负载均衡器，包括 ingress。这比较适合流量峰谷比较明显的场景

# 实际效果

- 测试环境节省超过 70%
- 生产环境待评估，预计超过 30%

# 相关资料

- [ECI 计费文档](https://help.aliyun.com/document_detail/154527.htm)
- ECS [价格表](https://www.aliyun.com/price/product#/ecs/detail/vm)
- 更多[注解说明](https://help.aliyun.com/document_detail/186939.html)， 如抢占式突发性能实例
