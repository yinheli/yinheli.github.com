---
title: postgresql 大版本升级
date: 2023-12-22
tags:
  - db
  - postgresql
  - upgrade
---
gitea 的 helm 升级连带升级了 postgresql , postgresql 的大版本升级比较麻烦，这里记录下。

- 首先把老的版本数据文件夹备份
- 老的 bin 文件夹备份
- 然后通过 `pg_upgrade` 来升级
- 拷贝一个老的或者新的 pod 的 yaml 配置，并且设置为 root 用户启动，command 改为 `tail -f /dev/null`

以上都做完，可以停掉老的服务，用以上备份的 k8s  pod yaml 改为新版本镜像，挂载老的数据盘，启动一个实例

```bash
# 创建 postgres 用户
adduser --uid 1001 postgres

# 老版本bin /bitnami/postgresql/oldbin
# 老版本data /bitnami/postgresql/olddata

mkdir /bitnami/postgresql/data

su postgres

# 初始化
initdb -E UTF8 -U postgres -D /bitnami/postgresql/data

# 升级
pg_upgrade -b /bitnami/postgresql/oldbin -B /opt/bitnami/postgresql/bin -d /bitnami/postgresql/olddata -D /bitnami/postgresql/data
```

如果此前的老版本没有正常退出，报错
```
The source cluster was not shut down cleanly
```

可以用老的 bin 重新启动关闭一下，确保它是正常的退出，然后在尝试执行上面的升级操作

```bash
/bitnami/postgresql/oldbin/pg_ctl start -w -D /bitnami/postgresql/olddata
/bitnami/postgresql/oldbin/pg_ctl stop -s -m fast -D /bitnami/postgresql/olddata
```

## 参考资料

- https://github.com/bitnami/charts/issues/8025