---
draft: false
date: 2015-01-20
title: "centos 6 安装 vmware tools 无法启动的问题"
description: ""
tags: ["centos"]
---

错误信息

```
vmware-tools-thinprint start/running
initctl: Job failed to start
Unable to start services for VMware Tools
```

尝试安装 `fuse-libs`

```
yum install fuse-libs
```

手工启动

```
/etc/vmware-tools/services.sh restart
```

```
Stopping VMware Tools services in the virtual machine:
   Guest operating system daemon:                          [确定]
   VMware User Agent (vmware-user):                        [确定]
   Blocking file system:                                   [确定]
   Unmounting HGFS shares:                                 [确定]
   Guest filesystem driver:                                [确定]
   VM communication interface socket family:               [确定]
   VM communication interface:                             [确定]
   Checking acpi hot plug                                  [确定]
Starting VMware Tools services in the virtual machine:
   Switching to guest configuration:                       [确定]
   VMware Automatic Kmods:                                 [确定]
   VM communication interface:                             [确定]
   VM communication interface socket family:               [确定]
   Guest filesystem driver:                                [确定]
   Mounting HGFS shares:                                   [确定]
   Blocking file system:                                   [确定]
   Guest operating system daemon:                          [确定]
```

问题解决
