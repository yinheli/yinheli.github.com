---
title: kubernetes node shell plugin
tags:
  - k8s
  - shell
  - kubectl
  - plugin
date: 2023-01-29
---

`kubectl exec` 可以进到 pod 容器内获取 shell，有时候需要 shell 到 node，而原生的 `kubectl debug` 虽然支持但是不支持特权模式，拿不到 root 权限，也不能用宿主机上的大部分工具，发现 `Lens` 上有这个功能，把核心的部分拷贝出来，做成脚本，作为 `kubectl` 插件。

将 `kubectl-shell` 放到 `PATH` 即可， `kubectl shell <node-name>`

```bash
vi kubectl-shell
```

```bash
#!/bin/bash
set -e

node=${1}
nodeName=$(kubectl get node ${node} -o template --template='{{index .metadata.labels "kubernetes.io/hostname"}}') 
nodeSelector='"nodeSelector": { "kubernetes.io/hostname": "'${nodeName:?}'" },'
podName=node-shell-${node}

kubectl run ${podName:?} --restart=Never -it --rm --timeout=5m --pod-running-timeout=5m --image overriden --overrides '
{
  "spec": {
    "hostPID": true,
    "hostNetwork": true,
    '"${nodeSelector?}"'
    "tolerations": [{
        "operator": "Exists"
    }],
    "containers": [
      {
        "name": "shell",
        "image": "alpine",
        "command": [
          "nsenter", "-t", "1", "-m", "-u", "-i", "-n",
          "bash"
        ],
        "stdin": true,
        "tty": true,
        "securityContext": {
          "privileged": true
        }
      }
    ]
  }
}' --attach "$@"
```

原理：

创建一个有特权模式的 pod，调度到对应的 node 上运行，然后 [`nsenter`](https://man7.org/linux/man-pages/man1/nsenter.1.html)  拿到 root shell


P.S.

发现已有社区相关[插件](https://github.com/kvaps/kubectl-node-shell)了，下次可以先去 [krew](https://krew.sigs.k8s.io/plugins/) 找一下 :)

