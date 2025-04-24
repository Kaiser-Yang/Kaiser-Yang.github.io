---
layout: post
title: gcs 部署
date: 2025-04-24 17:10:49+0800
last_updated: 2025-04-24 17:10:49+0800
description: 主要介绍如何部署 `gcs`。
tags:
  - Linux
  - Nginx
  - Docker
categories: Java
featured:
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

## 后端部署

从 [gcs-back-end](https://github.com/CMIPT/gcs-back-end/releases) 的 `Release` 页面下载最新的
`gcs-back-end.tar.gz`，该压缩包中包含了编译后的 `jar` 包以及相关的配置文件，目录结构如下：

```
.
├── .env
├── 3rdparty
├── Dockerfile
├── database
├── docker-compose.yml
├── nginx
├── start.sh
└── target
```

接下来需要修改 `.env` 文件，主要修改以下几个配置：

```bash
GIT_USER_PASSWORD=
SPRING_DRUID_PASSWORD=
GIT_SERVER_DOMAIN=
SPRING_MAIL_HOST=
SPRING_MAIL_PORT=
SPRING_MAIL_USERNAME=
SPRING_MAIL_PASSWORD=
SPRING_MAIL_PROTOCOL=
MD5_SALT=
JWT_SECRET=
GCS_SSH_MAPPING_PORT=10623
FRONT_END_REVERSE_PROXY_PORT=45698
```

其中 `JWT_SECRET` 以及 `MD5_SALT` 可以通过 `openssl rand -base64 32` 生成（请生成不同的值）。
`GCS_SSH_MAPPING_PORT` 和 `FRONT_END_REVERSE_PROXY_PORT` 设置成要开放的端口，
`GCS_SSH_MAPPING_PORT` 用于 `ssh` 克隆仓库，`FRONT_END_REVERSE_PROXY_PORT` 用于访问前端，
在只部署后端的情况下该变量无效。

**注意**：`MD5_SALT` 指定后不可以修改。

接下来你需要保证安装有 `openssl`, `docker` 和 `docker-compose`。

首先使用以下命令生成 `ssl` 自签名，如果你已经有了 `ssl` 证书，直接将证书放到 `nginx/ssl` 下，
并命名成 `private.key` 和 `certificate.crt`。

```bash
openssl req -x509 -nodes -days 36500 -newkey rsa:2048 -keyout nginx/ssl/private.key -out nginx/ssl/certificate.crt
```

接下来启动 `docker-compose`，依次执行：

```bash
docker-compose build
docker-compose up -d
```

此时后端成功在后台启动，端口为 `8080`。
