---
title: kaniko build cli
tags: [k8s, image, kaniko]
date: 2023-01-18
---

造了个 cli 工具，[kaniko-build](https://github.com/yinheli/kaniko-build) 用于在 k8s 集群中打包镜像。

kaniko 是我们用的比较多的工具，包括 tekton 流水线上也在使用，如果在本地调试，此前要么是本地 docker build 要么就是几个 bash 脚本，构建环境和创建构建资源，这次把这些脚本整理了一下，写成 python 脚本，用 `click` 这个包很快就做成一个 cli 工具，而相关的 yaml 文件则使用模板渲染丢给 kubectl pipe，觉得清爽方便了很多。

借助 kaniko 本身的能力，它除了可以支持挂载 pv ，使用本地 Dockerfile， 也支持远程包括 git， s3 等，具体请参考官方文档吧，我只用到 pv 挂载和 git 远程库两种场景，其他场景未做测试。

# 相关资料
- kaniko [https://github.com/GoogleContainerTools/kaniko](https://github.com/GoogleContainerTools/kaniko)

