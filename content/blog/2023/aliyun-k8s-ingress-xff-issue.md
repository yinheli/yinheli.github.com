---
title: 阿里云 k8s ingress xff 问题
tags: [k8s, aliyun, xff]
date: 2023-01-17
---

在排查 IP 问题的时候发现记录的 IP 有明显问题，都是运营商或者机房的，感觉 xff 出问题了，修改 nginx-ingress 的 ConfigMap 解决，以下为推荐配置，请参考官方文档和业务情况适当调整
```yaml
generate-request-id: "true"
use-forwarded-headers: "true"
enable-real-ip: "true"
proxy-real-ip-cidr: "0.0.0.0/0"
proxy-add-original-uri-header: "true"
compute-full-forwarded-for: "true"
allow-backend-server-header: "true"
enable-underscores-in-headers: "true"
```

默认的 ingress 对应的负载均衡器为安装时的固定规格，也可以参考 Annotation 配置，调整 nginx-lb 的 service，调整规格，计划方案，如 LCU 计费或包年包月计划等，如果波峰比较大，个人比较推荐 LCU 的计费模式，省钱的同时也能应对突增流量。

## 相关资料
- https://github.com/kubernetes/ingress-nginx
- https://kubernetes.github.io/ingress-nginx/
- https://help.aliyun.com/document_detail/86531.html