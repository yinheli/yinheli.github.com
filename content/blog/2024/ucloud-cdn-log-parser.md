---
title: ucloud CDN 日志解析
tags:
  - cloud
  - cdn
  - ucloud
  - rust
date: 2024-04-20
---
前几天遇到 ucloud CDN 服务的带宽报警，而他们的后台对流量分析这块太弱了，于是看了下文档对照写了个[日志解析](https://github.com/yinheli/ucloud-cdn-log-parser)，主要是练手 rust，又有一年多没有做 rust 的项目了。

这个工具只是简单的用正则对日志文件做了解析，具体的分析需要再配合其他工具处理，我这里比较推荐使用 duckdb 或者 clickhouse (local)。将数据解析转换为 parquet 后压缩率很高。

顺便发现 duckdb 的 csv/tsv 数据导入 parquet 比 clickhouse 支持的更好，感觉数据类型的推理更智能。

具体使用姿势参考项目的 [readme](https://github.com/yinheli/ucloud-cdn-log-parser?tab=readme-ov-file#usage) 。


