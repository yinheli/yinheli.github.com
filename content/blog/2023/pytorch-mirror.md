---
title: pytorch 镜像
tags: [pytorch, nexus]
date: 2023-08-08
---

pytorch 貌似并未提供官方的 pypi 仓库，仅有一个 wheels 下载站 https://download.pytorch.org/whl ,当然 pip 是可以识别 `find-links` 类型的，但是在国内的网络环境下，下载却非常慢，尝试用上海交大的镜像，能临时解决问题，不过我们要在项目中构建 Docker 镜像，还是用免费的 nexus 私服构建代理比较合适。

而 nexus 私服的 pypi 只支持代理 pypi 的上游，不支持 `find-links` 规范，故曲线救国，先手动把 whl 包下载下来，再用 `twine` 上传/发布到私服。

先到华中科大的[镜像站](https://mirror.sjtu.edu.cn/pytorch-wheels/cu118/?mirror_intel_list)下载需要的 whl 包 , 留意下载地址的那个参数，必传否则跳转到官方，可能机器学习大热华中科大的流量也是比较惊人吧。

1. 在浏览器的 console 执行下面脚本，拷贝所有 linux 的发行包（按照你的需要调整） ```

```js
copy($$('a').map(it=>it.href).filter(it=>it.indexOf('linux')>0).join('\n'))
```

2. 配置 `~/.pypirc`

```ini
[distutils]
index-servers =
    nexus-torch-cu118

[nexus-torch-cu118]
repository = https://nexus-server/repository/pytouch-cu118/
username = user
password = pass
```

3. 安装好 `twine` ，挨个包执行上传/发布

```bash
twine upload --repository nexus-torch-cu118 torch-2.0.0+cu118-cp39-cp39-linux_x86_64.whl
```

4. 记得在 pypi group 里加上发布的那个 self-host 仓库


