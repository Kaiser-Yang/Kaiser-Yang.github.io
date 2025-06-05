---
layout: post
title: gcs-back-end Development
date: 2025-06-04 20:04:58+0800
last_updated: 2025-06-04 20:04:58+0800
description: This post includes the main process of gcs-back-end development.
tags:
  - Docker
  - Spring
  - Backend
  - Java
categories: gcs
featured:
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

The main process of `gcs-back-end` development is as follows:

* Clone the `gcs-back-end` repository.
* Write the code.
* Compile and test the code.
* Commit the code to the repository.
* Open a pull request to the `gcs-back-end` repository.
* Wait for the code review and merge.

## Clone the Repository

If you want to run the project locally, you should clone the sub-modules, too.
Use the following command to clone the repository and its sub-modules:

```bash
# HTTPS
git clone --recursive https://github.com/CMIPT/gcs-back-end.git
# or with SSH
git clone --recursive git@github.com:CMIPT/gcs-back-end.git
```

If you have cloned the repository before, you just need to update the sub-module:

```bash
# Enter the gcs-back-end root first
git submodule init
git submodule update
```

## Write the Code

Then you should write the code related to the feature you want to implement
or the bug you want to fix.

Do not forget to write the integration tests for the code you write, which is required.

## Compile and Test the Code

If you just want to compile, make sure you have `mvn` and `jdk17` (or later) installed on your system.

If you want to run the tests, you should have a database set up. We recommend you to
use `docker-compose` to run the tests. Make sure `mvn`, `jdk17` (or later), `docker`, `openssl`,
and `docker-compose` are installed on your system.

Or if you do not want to compile and test the code locally,
you can skip this step. When you open a pull request to the `master` branch of the repository,
GitHub Actions will automatically build the project and run the tests.

### Compile

Use the following command to compile the project:

```bash
mvn compile
```

### Test

For test, you should build the `jar` package first:

```bash
# Skip tests, as we have no database to run tests
mvn package -Dmaven.test.skip=true
```

Once it successfully built, you will find the `jar` package in the `target` directory.

The second step is to configure the deployment environment.
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
`GCS_SSH_MAPPING_PORT` is the ports you want to expose,
which is used for `ssh` cloning repositories,

**NOTE**: `MD5_SALT` should not be changed after it is set.

Now you can use the command below to start the `gcs-back-end` service:

```bash
docker-compose build
docker-compose up -d
```

Now the `gcs-back-end` service is running in the background listening on port `8080`.

You can use `docker ps` to check if the service is running (named with `gcs`).

Once the service is running, you can access the API documentation at:

`http://localhost:8080/swagger-ui/index.html`

If you want to update the `jar` package, you can use the following command:

```bash
# Rebuild the jar package
mvn package -Dmaven.test.skip=true
# Substitute the old jar package with the new one
docker cp target/gcs-back-end.jar gcs:/gcs
# Restart the gcs service
docker restart gcs
```

Now you can copy the whole repository to the `gcs` docker
so that you can run the tests in the docker container:

```bash
docker cp . gcs:/root/gcs-back-end
```

Now you can enter the `gcs` docker container and run the tests:

```bash
# Enter the gcs docker container
docker exec -it gcs /bin/bash
# Install maven
sudo apt install -y maven
# Enter the gcs-back-end directory
cd /root/gcs-back-end
# Run the tests
mvn test
```

Once the tests are passed, you can exit the docker container.

The next time you want to run the tests, you just need use those commands:

```bash
# Update the source code in docker container
docker cp . gcs:/root/gcs-back-end
# Enter the gcs docker container
docker exec -it gcs /bin/bash
# Enter the gcs-back-end directory
cd /root/gcs-back-end
# Run the tests
mvn test
```

## Commit the Code

Now you can commit the code to the repository:

```bash
git add .
git commit -m "Your commit message"
# Or you can use your own GUI to write the commit message
```

The commit message should be clear and concise, those below are some examples:

```Bash
# Example 1
fix(repository): empty repository name

In this commit, we use @NotBlank to ensure the repository name
is not empty. And add NOT NULL constraint to the repository
name in the database. This should close #1234.

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
be less than `72` characters.

You should try to make sure the commits are atomic, meaning that each commit should
only contain one logical change. And you should try to make every commit can
be built and tested successfully.

## Open a Pull Request

Once you have committed the code, you can push the code to your forked repository, or if you are
one of the collaborators, you can push the code to the `gcs-back-end` repository directly.

Then open a pull request to the `master` branch of the `gcs-back-end` repository.

The title and description of the pull request should be clear and concise,
you can pick up the main commit message as the title of the pull request.

## Wait for Code Review and Merge

You can send a message to the `gcs-developers` team to notify them that
you have opened a pull request.

Once the pull request is opened, the team will review your code and give you feedback.

During this process, you may be asked to make some changes to your code.

Once the code is reviewed and approved, the team will merge your code to the `master` branch.

