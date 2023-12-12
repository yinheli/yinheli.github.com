---
title: 本地运行 loki 查询
tags:
  - loki
  - docker
date: 2023-12-12
---
loki 的 [LogQL](https://grafana.com/docs/loki/latest/query/) 使用体验还是很好的，一方面能过滤查询，另一方面还能看到日志的趋势。

我的场景是要临时分析 CDN 的某个域名的日志

## 准备 docker compose 运行环境

```bash
vi loki.yaml
```

```yaml
auth_enabled: false

server:
  http_listen_port: 3100
  grpc_server_max_recv_msg_size: 41943040
  grpc_server_max_send_msg_size: 41943040

common:
  path_prefix: /loki
  storage:
    filesystem:
      chunks_directory: /loki/chunks
      rules_directory: /loki/rules
  replication_factor: 1
  ring:
    kvstore:
      store: inmemory

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h
limits_config:
  reject_old_samples: false
  ingestion_rate_mb: 1024
  per_stream_rate_limit: 512MB
```

```bash
vi docker-compose.yaml
```

```yaml
version: "3"

services:
  loki:
    image: docker.mirrors.sjtug.sjtu.edu.cn/grafana/loki
    user: root
    volumes:
      - /data/loki:/loki
      - ./loki.yaml:/etc/loki/local-config.yaml
    ports:
      - "3100:3100"

  grafana:
    image: docker.mirrors.sjtug.sjtu.edu.cn/grafana/grafana
    ports:
      - "3000:3000"
```

## 导入日志

可以用 promtail 或者如果数据有自己的预处理需求，就直接写个脚本，我这里写的 python 脚本来处理

```python
import gzip
import json
import os

import httpx

# https://console.bce.baidu.com/cdn/#/cdn/customlog
fields = 'remote_addr remote_ip remote_port jvip time_local scheme host format_time request status bytes_sent body_bytes_sent sent_http_content_length http_content_length request_time request_time_ms connection udf_hit http_range http_referer http_cookie http_user_agent http_x_forwarded_for ssl_protocol sent_http_content_type sent_http_content_range server_protocol http_accept log_level req_len url request_method uri http_ver log_timestamp_ms log_timestamp_s'.split(' ')

def iter_log(batch=1000):
    items = []
    # list dir gzip iter all gz files
    for file in sorted(os.listdir('.')):
        if file.endswith('.gz'):
            with gzip.open(file, 'rt') as f:
                for line in f:
                    line = line.strip()
                    if not line:
                        continue
                    line = line.split('\t')
                    if len(line) != len(fields):
                        continue
                    item = dict(zip(fields, line))
                    items.append([str(int(item['log_timestamp_ms']) * 1000000), json.dumps(item, ensure_ascii=False)])
                    if len(items) == batch:
                        yield items
                        items = []
    if items:
        yield items


# https://grafana.com/docs/loki/latest/reference/api/
for items in iter_log(batch=1000):
    data = {
        "streams": [
            {
                "stream": {
                    "cdn": "baidu"
                },
                "values": items
            }
        ]
    }
    r = httpx.post('http://127.0.0.1:3100/loki/api/v1/push', json=data)
    if r.status_code != 204:
        print(r.status_code, r.text)
```