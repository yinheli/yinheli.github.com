---
title: nginx url decode issue
tags: [nginx, njs, bitbucket]
date: 2023-02-07
---

公司的 bitbucket 已使用多年，最近计划升级，主要期望解决 nextjs 项目中文件名带有括号时 bitbucket 无法在 web 端查看代码的问题，发现升级完成后问题依旧，而相同版本，本地实验环境则没有发现这个问题，突然想到是不是中间代理层的问题，直连后 bitbucket (tomcat) 服务后发现果然没有这个问题，没想到困扰了这么久的问题居然是 nginx 导致的……

尝试了多种方式，包括 rewrite 等，`proxy_pass` 依旧转发的内容会先做 url decode，这导致了后端server无法处理，最后本想用 lua 来试试，不过代理网关那边是 `apt` 直接安装的 nginx，改为 openresty 改动还有点大，遂试试 apt 安装 [`nginx-module-njs`](https://nginx.org/en/docs/njs/install.html).

修改 nginx.conf, 添加 `ngx_http_js_module` 加载

```conf
load_module /usr/lib/nginx/modules/ngx_http_js_module.so;
```

修改有关 bitbucket 段的反向代理配置，导入 git.js，用 auth_request 每次请求都执行 `/urlencode` 块 

```conf
js_import /etc/nginx/conf.d/git.js;
js_set $plain_uri 'empty';

location ^~ / {
  # ...
  auth_request /urlencode;
  proxy_pass http://10.180.9.229:7990$plain_uri;
}

location /urlencode {
  internal;
  js_content git.urlencode;
}
```

其中 `git.js` 如下

> 这里没有用 `replaceAll` 是因为在 `njs` 中并不支持

```js
function urlencode(r){
  let uri = r.variables.request_uri;
  r.variables.plain_uri = uri.replace(/\[/g, '%5B').replace(/\]/g, '%5D');
  r.return(200);
}
export default {urlencode};
```

