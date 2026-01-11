---
layout: post
title: Deploy gcs
date: 2025-04-24 17:10:49+0800
last_updated: 2025-06-04 19:59:07+0800
description: This post introduces how to deploy gcs on a server using Docker.
tags:
  - English Posts
  - Linux
  - Nginx
  - Docker
categories: gcs
featured:
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

We only recommend deploying `gcs` by docker compose, which is the easiest way.

## Prerequisites

* `openssl`
* `docker`
* `docker-compose`

## Deployment

You should download the latest release from
[gcs-front-end](https://github.com/CMIPT/gcs-front-end/releases).

The downloaded file is a compressed file named `gcs.tar.gz`,
which contains the compiled `jar` package, `js` and `css` files, and related configuration files.

The directory structure is as follows:

```
.
├── .env
├── .output
├── 3rdparty
├── Dockerfile
├── database
├── docker-compose.yml
├── nginx
├── start.sh
├── target
```

Then you need to do some configurations for the deployment environment.
You mainly need to modify the `.env` file in the root directory of the project.
Those below are the environment variables you need to set:

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
GCS_SSH_MAPPING_PORT=
FRONT_END_REVERSE_PROXY_PORT=
```

You can generate `JWT_SECRET` and `MD5_SALT` using the command `openssl rand -base64 32`
(make sure they are different).
`GCS_SSH_MAPPING_PORT` and `FRONT_END_REVERSE_PROXY_PORT` are the ports you want to expose.
`GCS_SSH_MAPPING_PORT` is used for `ssh` cloning repositories.
`FRONT_END_REVERSE_PROXY_PORT` is used for accessing the front end.

**NOTE**: `MD5_SALT` should not be changed after set.

Then, you should copy your certificates to make sure `HTTPS` works properly.
You can use your own `ssl` certificate and make sure they are placed in the `nginx/ssl` directory
with names `private.key` and `certificate.crt`.

Or you can generate the `ssl` certificate by running the following command:

```bash
openssl req -x509 -nodes -days 36500 -newkey rsa:2048 \
  -keyout nginx/ssl/private.key -out nginx/ssl/certificate.crt
```

Now you can use the command below to start the `gcs` service:

```bash
docker-compose build
docker-compose up -d
```

After this, the `gcs` service is running in the background,
you can browse the front end at `https://your-domain:FRONT_END_REVERSE_PROXY_PORT`.

