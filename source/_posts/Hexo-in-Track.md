---
title: Hexo in Track
date: 2024-03-07 11:41:26
tags:
categories:
---
# 网站维护
## HTTPS 服务
当前采用的解决方案是 `certbot` + `nginx`

首先需要捆绑域名，dig 后

- `trojan` 需要手动配置，配置文件如下：
    - cert, key 搭配 let's encrypt 时， 默认的在 "/etc/letsencrypt/live" 下，
```
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 8964,
    "remote_addr": "127.0.0.1",
    "remote_port": 80,
    "password": [
        "neverknowsbest"
    ],
    "log_level": 1,
    "ssl": {
        "cert": "cert.pem",
        "key": "key.pem"
    }
}
```

- `certbot` 用来部署 ssl， 一键式解决 `certbot --nginx`

- `nginx` 需要初始化，可以从网上拷贝一份初始化设置，我昨天倒腾了一下，我也不知道到底怎么搞的。