---
layout: post
title: gcs-front-end Development
date: 2025-06-05 09:26:27+0800
last_updated: 2025-06-05 09:26:33+0800
description: This post includes the main process of gcs-front-end development.
tags:
  - Frontend
  - Vue
categories: gcs
featured:
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

The main process of `gcs-front-end` development is as follows:

* Clone the `gcs-front-end` repository.
* Code and test.
* Commit the code to the repository.
* Open a pull request to the `gcs-front-end` repository.
* Wait for the code review and merge.

## Clone the Repository

```bash
# HTTPS
git clone https://github.com/CMIPT/gcs-front-end.git
# or with SSH
git clone git@github.com:CMIPT/gcs-front-end.git
```

## Code and Test

You may need to deploy the `gcs-back-end` first,
so that you can request APIs from the front-end.

We recommend you to use the `docker-compose` to deploy the `gcs-back-end`, those below are required:

* `docker`
* `docker-compose`
* `openssl`

First, you should download the latest release from
[gcs-back-end](https://github.com/CMIPT/gcs-back-end/releases).

The downloaded file is a compressed file named `gcs-back-end.tar.gz`,
which contains the compiled `jar` package, and related configuration files.

The directory structure is as follows:

```
.
├── .env
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
GIT_SERVER_DOMAIN=
SPRING_MAIL_HOST=
SPRING_MAIL_PORT=
SPRING_MAIL_USERNAME=
SPRING_MAIL_PASSWORD=
SPRING_MAIL_PROTOCOL=
MD5_SALT=
JWT_SECRET=
GCS_SSH_MAPPING_PORT=
```

You can generate `JWT_SECRET` and `MD5_SALT` using the command `openssl rand -base64 32`
(make sure they are different).
`GCS_SSH_MAPPING_PORT` is the port you want to expose, which is used for `ssh` cloning repositories.

**NOTE**: `MD5_SALT` should not be changed after set.

You can generate the `ssl` certificate by running the following command:

```bash
openssl req -x509 -nodes -days 36500 -newkey rsa:2048 \
  -keyout nginx/ssl/private.key -out nginx/ssl/certificate.crt
```

Now you can use the command below to start the `gcs` service:

```bash
docker-compose build
docker-compose up -d
```

After this, the `gcs` service is running in the background listening on port `8080`.

Once the service is running (you can check the status by running `docker ps`),
you can access the API documentation at:

`http://localhost:8080/swagger-ui/index.html`

Now it's time for you to code and test.

## Commit the Code

After you finish your coding and testing, you can commit the code to the repository:

```bash
git add .
git commit -m "Your commit message"
# Or you can use your own GUI to write the commit message
```

The commit message should be clear and concise, those below are some examples:

```Bash
# Example 1
fix(repository): empty repository name

In this commit, we use asynchronous request to check if the repository
name is not empty. This should close #1234.

# Example 2
ci(format): format ci pipeline

We add new github actions to format the code automatically when
a pull request is opened.
```

The first line of the commit message should started with the type of the commit,
which can be one of the following:

* `feat`: a new feature
* `fix`: a bug fix
* `docs`: documentation only changes
* `format` / `style`: formatting changes
* `refactor`: a code change that neither fixes a bug nor adds a feature
* `perf`: a code change that improves performance
* `test`: adding missing or correcting existing tests
* `ci`: changes to our CI configuration files and scripts
* `build` / `deps`: changes that affect the build system or external dependencies

Then you should add the affected modules in the parentheses, then a colon `:`, and
a brief description of the change.

After the first line, you should leave a blank line,
then write a detailed description of the change. Every line of the description should
not be greater than `72` characters.

You should try to make sure the commits are atomic, meaning that each commit should
only contain one logical change. And you should try to make every commit can
be built and tested successfully.

## Open a Pull Request

Once you have committed the code, you can push the code to your forked repository, or if you are
one of the collaborators, you can push the code to the `gcs-front-end` repository directly.

Then open a pull request to the `master` branch of the `gcs-front-end` repository.

The title and description of the pull request should be clear and concise,
you can pick up the first line of your commit message as the title of the pull request.

## Wait for Code Review and Merge

You can send a message to the `gcs-developers` team to notify them that
you have opened a pull request.

Once the pull request is opened,
the team will review your code and give you feedback as soon as possible.

During this process, you may be asked to make some changes to your code.

Once the code is reviewed and approved, the team will merge your code to the `master` branch.
