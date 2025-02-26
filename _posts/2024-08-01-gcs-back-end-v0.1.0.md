---
layout: post
title: gcs-back-end v0.1.0 开发实录
date: 2024-08-12 11:59:00+0800
description:
tags:
  - Spring
  - Spring MVC
  - Spring Boot
categories: Java
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

NOTE: 文章内容按照时间 `pr` 创建的时间线进行排列。

仓库地址：[gcs-back-end](https://github.com/CMIPT/gcs-back-end)。

# Add docker creator and format action

`pr`的链接：[gcs-pull-1](https://github.com/CMIPT/gcs-back-end/pull/1)

## Google Java Style Format

本次`pr`添加了`docker`的创建和格式化`action`。该`action`能够在创建合入`master`和`develop`的`pr`时，
自动使用`Google Java Style Format`对`Java`代码进行格式化，
并且自动创建一个新的`commit`提交格式化后的代码。

创建这个`git action`的主要目的是为了保证整个团队编码风格的一致性。

Issues:

- [ ] 这个`action`目前还有一些问题，比如新创建的提交依然会触发一次该`action`。

NOTE: `git action`的代码中出现了`secrets.PAT`，该字段需要在仓库的设置部分自行进行设置，
值的内容是一个`token`，该`token`具有仓库的写权限，这样自动提交的时候才能保证能够成功提交。
而`token`的创建方法是在个人的`github`账号中进行创建。

在`github`中创建`token`：

![](https://raw.githubusercontent.com/Kaiser-Yang/image-hosting-site/main/20240421-20250421/20240801102637.png){: .img-fluid}

如何在仓库的`secrets`中添加`token`：

![](https://raw.githubusercontent.com/Kaiser-Yang/image-hosting-site/main/20240421-20250421/20240801102200.png){: .img-fluid}

## Docker Creator

本次`pr`中还添加了一个第三方依赖，用于创建`docker`镜像。
这个镜像能够自动创建一个`docker`镜像并安装一些基础的工具，例如`ssh`等。

第三方依赖的地址：[docker-creator](https://github.com/CMIPT/docker-script.git)

# Initialize the project

`pr`的链接：[gcs-pull-2](https://github.com/CMIPT/gcs-back-end/pull/2)

本次`pr`是从[spring-io](https://start.spring.io/)上创建了一个`Spring Boot`的项目，
然后将其解压到了仓库中。

TODO:

- [x] 添加`spring`的基本使用介绍

# Build the initial database script

`pr`的链接：[gcs-pull-3](https://github.com/CMIPT/gcs-back-end/pull/3)

# Provide Python scripts to process `Json` files

`pr`的链接：[gcs-pull-6](https://github.com/CMIPT/gcs-back-end/pull/6)

本次的`pr`提供了处理`Json`文件的`Python`脚本，脚本接受一个参数，表示文件所在目录。
默认值为`../config.json`。脚本内置函数将`Json`文件的内容值存入对象`a`，
并可以通过点操作符`.`访问对象属性来获取对应内容值。

```python
def loadJsonAsObject(file_path: str):
    with open(file_path, 'r', encoding='utf-8') as file:
        data = json.load(file)
    # transmit dictronary into object
    return json.loads(json.dumps(data), object_hook=lambda d: SimpleNamespace(**d))


parser = argparse.ArgumentParser(description="Process JSON file.")
parser.add_argument('file_path', nargs='?', default='../config.json', help="Path to the JSON file")
args = parser.parse_args()
```

下面介绍一下上述代码使用的`argparse`功能：

* `argparse.ArgumentParser`：该类用于创建一个参数解析器对象。
`description`参数可以用于设置命令行工具的简短描述。
* `parser.add_argument`：定义程序可以接受的命令行参数。
    - `file_path`：这是定义的命令行参数名。它将接受用户输入的文件路径。
    - `nargs='?'`：表示该参数是可选的。如果用户没有提供该参数，则使用默认值。
    - `default='../config.json'`：如果用户没有提供`file_path`参数，则默认使用`../config.json`作为文件路径。
    - `help="Path to the JSON file"`：提供该参数的帮助信息，当用户使用`--help`选项时会显示这些信息。
* `args = parser.parse_args()`：解析命令行参数，并将结果存储在`args`对象中。

函数`loadJsonAsObject()`使用`json.loads()`时默认将内容解析成字典对象`dict`，
使用`SimpleNamespace`参数后，允许像访问对象属性一样访问字典键。

# Add spring-doc for restful api

`pr`的链接：[gcs-pull-7](https://github.com/CMIPT/gcs-back-end/pull/7)

本次的`pr`主要是添加了`spring-doc`的依赖，用于生成`restful api`的文档。

集成文档的时候开始在尝试使用`spring-fox`和`spring-swagger`。
结果发现和`spring3.x`的版本不兼容，然后搜索资料发现了`spring-doc`这个依赖，然后就使用了这个依赖。

- [x] 添加`spring-doc`的使用介绍。

# Finish part of the deploy script

`pr`的链接：[gcs-pull-9](https://github.com/CMIPT/gcs-back-end/pull/9)

创建这个脚本的目的是希望能够仅仅通过编写一个`json`的配置文件，就能够完成整个项目的部署。
这个`pr`完成了部分的功能，还有一些功能没有完成。

主脚本是一个`bash`脚本，在`bash`脚本中会自动安装`python`等依赖，然后调用`python`脚本来完成部署。
部署的主要逻辑是通过读取`json`文件，然后根据配置文件完成一系列处理，
最后将`jar`通过`Sys V init`注册成一个服务。

`Sys V init`创建服务的方式非常简单，只需要在`/etc/init.d/`目录下放置一个可执行文件，然后即可通过
`service xxx start`来启动服务。在本次的`pr`中，直接创建了一个软连接连接到了打包出来的`jar`文件，
但是在后续的使用过程中发现这样做存在问题。

在本次的提交中，我发现`junit`可以使用`spring-boot-test`进行替代，于是将之前的`junit`的依赖替换成了
`spring-boot-test`，并且删除了之前添加的`log4j`依赖（后续可以自己设置`spring-boot`的日志系统）。

下面介绍一下几个脚本有什么作用：

* `deploy_ubuntu.sh`：这个脚本是主脚本，用于在`ubuntu`系统上部署项目，
这个脚本在安装一些依赖后便将控制权交给`script/deploy_helper.py`。
* `script/deploy_helper.py`：这个脚本读取`json`配置文件，然后根据配置文件完成一系列操作，
最后将`jar`注册成一个服务。
* `script/get_jar_position.sh`：获取`jar`的位置，这个脚本会在`deploy_helper.py`中被调用，
用于获取`mvn package`输出位置。

Issues:

- [x] 这个脚本在某些环境使用的时候出现了问题，详见[gcs-issues-14](https://github.com/CMIPT/gcs-back-end/issues/14)。
- [x] 错误的添加使用了`spring-boot-test`导致添加了`spring-boot-starter-webflux`依赖，这个依赖目前应该是不用的。

# Add MIT license and developers info

`pr`的链接：[gcs-pull-13](https://github.com/CMIPT/gcs-back-end/pull/13)

本次`pr`主要是添加了`MIT`的开源协议，以及开发者的信息。

# Substitute the Sys V init with systemd

`pr`的链接：[gcs-pull-15](https://github.com/CMIPT/gcs-back-end/pull/15)

本次`pr`主要是将`Sys V init`部署服务的方式替换成了`systemd`。
这样做是因为发现现代的`Linux`系统更多地使用`systemd`来管理服务，且`systemd`的启动速度等更加优秀，
配置更加简单。这次的提交修复了[gcs-issues-14](https://github.com/CMIPT/gcs-back-end/issues/14)。

这次的提交中还增加了`config_default.json`文件，`config_default.json`文件存储默认的配置，
当一个配置项没有被用户重写的时候会使用默认的配置。

同时新增了一个`clean_ubuntu.sh`脚本用于清空创建的服务等。

下面简单介绍一下`systemd`的使用方法。

要使用`systemd`来管理服务，首先需要创建一个`service`文件，
然后将这个文件放置在`/etc/systemd/system/`目录下。`service`文件的内容大致如下：

```shell
[Unit]
Description=Git server center back-end service
After=network.target

[Service]
PIDFile=/var/run/gcs.pid
User=gcs
WorkingDirectory=/opt/gcs
Restart=always
RestartSec=5
ExecStart=/usr/bin/java -jar /opt/gcs/gcs.jar

[Install]
WantedBy=multi-user.target
```

上面的大部分字段是不用解释的，这里给出一些注意事项：

* 路径全部使用绝对路径。
* `systemctl enable gcs`表示将服务设置为开机启动，但是如果当前的服务并没有启动，那么这个命令并不会启动服务。
* `systemctl start gcs`表示启动当前的服务，这与`gcs`是否被设置为开机启动无关。
* `systemctl disable gcs`和`systemctl stop gcs`的关系与上述类似。

NOTE: 简单解释一下`systemctl enable gcs`和`systemctl disable gcs`的原理。两者只做了一件事情：
根据`[Install]`部分的内容创建或者删除软连接，例如上面的例子中，`systemctl enable gcs`会在
`/etc/systemd/system/multi-user.target.wants/`目录下创建一个`gcs.service`的软连接，而
`systemctl disable gcs`会删除这个软连接。
这个软连接的作用是在`multi-user.target`启动的时候启动`gcs`
而`Linux`系统启动的时候会启动`multi-user.target`，所以`gcs`也能在开机的时候自动启动。

# Add command_checker and setup_logger to script/deploy_helper.py

`pr`的链接：[gcs-pull-17](https://github.com/CMIPT/gcs-back-end/pull/17)

本次`pr`修改了`script/deploy_helper.py`，增加了日志功能。在执行`deploy_helper.py`时，如果命令执行失败，
会输出执行失败的命令的日志信息，包括时间，日志等级，行号，执行失败的命令。

```python
import logging
import inspect
```

首先，实现这个`pr`的功能，需要导入`logging`和`inspect`模块。这两个模块是`Python`内置模块，
在这里用于获取命令的行号。

```python
def setup_logger(log_level=logging.INFO):
    """
    Configure the global logging system.

    :param log_level: Set the logging level, defaulting to INFO.
    """
    logging.basicConfig(level=log_level,
                        format='%(asctime)s -%(levelname)s- in %(pathname)s:%(caller_lineno)d: %(message)s', 
                        datefmt='%Y-%m-%d %H:%M:%S')
```

`setup_logger`函数用于配置全局的日志系统。`log_level`参数用于设置日志级别，默认为`INFO`。
`logging.basicConfig`定义输出日志的格式，包括时间，日志等级，行号，执行失败的命令。

```python
def command_checker(status_code: int, message: str, expected_code: int = 0):
    """
    Check if the command execution status code meets the expected value.

    :param status_code: The actual status code of the command execution.
    :param message: The log message to be recorded.
    :param expected_code: The expected status code, defaulting to 0.
    """
    if status_code != expected_code:
        caller_frame = inspect.currentframe().f_back
        logging.error(message, extra={'caller_lineno': caller_frame.f_lineno})
        exit(status_code)
```

`command_checker`用于比较命令执行返回的状态码与期望的状态码是否一致，如果不一致，则说明命令执行失败，
则打印出相应的日志，且返回状态码，如果一致则说明命令执行成功，不做任何操作。

- `status_code`: 命令执行的实际状态码
- `message`: 要打印的日志信息
- `expected_code`: 期望的状态码，默认为0

```python
message_tmp = '''\
The command below failed:
    {0}
Expected status code 0, got status code {1}
'''
```

`message_tmp`是一个模板字符串，用于格式化输出日志信息，在这里会将执行失败的命令和状态码输出到日志中。

# Remove unused dependency and add doc for configuration

`pr`的链接：[gcs-pull-22](https://github.com/CMIPT/gcs-back-end/pull/22)

本次`pr`主要是将`spring-boot-starter-webflux`依赖删除，因为这个依赖是多余的。
同时添加了一个`README-zh.md`文件，用于存储配置文件的说明。

# Refactor the database script

`pr`的链接：[gcs-pull-23](https://github.com/CMIPT/gcs-back-end/pull/23)

在本次提交中，对数据库脚本进行了重构，将一个`sql`脚本分割成多个功能不同的`sql`脚本并放入不同的目录当中，
降低了数据库代码的耦合度，方便后续更新。并且还提供了一个数据库部署脚本`database_deploy.sh`，
运行这个脚本就能够自动调用前面的`SQL`脚本，从而部署数据库。

目前`database`的目录结果如下所示：

![](https://typora-picture-cloud.oss-cn-chengdu.aliyuncs.com/img1/202408191959133.png){: .img-fluid}

在`constraint/all_column_constraint.sql`文件中，定义了表的主键和唯一键约束；
`update_gmt_updated_column`文件包含一个函数，用于在更新时自动将`gmt_updated`列设置为当前的时间戳；
`sequence/all_column_seq.sql`文件和`sequence/sequence_set.sql`为表的主键列定义了序列以及设定列的当前值；
在`table`目录下面的三个文件定义了三个表，并且为每列添加了注释；
`trigger/all_table_trigger.sql`文件为表添加了触发器，自动在更新行的时候更新`gmt_updated`列。

# Finish the script for deploying in docker

`pr`的链接：[gcs-pull-24](https://github.com/CMIPT/gcs-back-end/pull/24)

在本次的提交中，增加了自动在 `docker` 中部署的功能。在编写这部分功能的时候，
发现 `docker` 在默认情况下是不能够使用 `systemd` 的，
只有当指明 `--privileged=true` 的时候才能够使用 `systemd`。
这个参数的作用是让 `docker` 在容器中运行的时候拥有直接操作宿主机的权限。
如果通过样的方式创建 `docker` 失去了 `docker` 的部分安全性，
因此我将在 `docker` 中的部署改用成了 `Sys Init V` 的方式，而在物理机上的部署继续保持`systemd` 的方式。

`Sys Init V` 的脚本模板来自于 [_service.md](https://gist.github.com/naholyr/4275302) 。
我对其中进行了部分的修改，得到了如下的文件：

```bash
PIDDIR=$(dirname "$PIDFILE")
LOGDIR=$(dirname "$LOGFILE")
start() {
  if [ -f "$PIDDIR/$PIDNAME" ] && kill -0 "$(cat "$PIDDIR/$PIDNAME")"; then
    echo 'Service already running' >&2
    return 1
  fi
  echo 'Starting service…' >&2
  local CMD="$SCRIPT &> \"$LOGFILE\" & echo \$!"
  su -c "mkdir -p ""$PIDDIR" "$RUNAS"
  su -c "mkdir -p ""$LOGDIR" "$RUNAS"
  su -c "$CMD" "$RUNAS" > "$PIDFILE"
  echo 'Service started' >&2
}

stop() {
  if [ ! -f "$PIDFILE" ] || ! kill -0 "$(cat "$PIDFILE")"; then
    echo 'Service not running' >&2
    return 1
  fi
  echo 'Stopping service…' >&2
  kill -15 "$(cat "$PIDFILE")" && rm -f "$PIDFILE"
  echo 'Service stopped' >&2
}

uninstall() {
  echo -n "Are you really sure you want to uninstall this service? That cannot be undone. [yes|No] "
  local SURE
  read SURE
  if [ "$SURE" = "yes" ]; then
    stop
    rm -f "$PIDFILE"
    echo "Notice: log file is not be removed: '$LOGFILE'" >&2
    update-rc.d -f "$NAME" remove
    rm -fv "$0"
  fi
}

case "$1" in
  start)
    start
    ;;
  stop)
    stop
    ;;
  uninstall)
    uninstall
    ;;
  restart)
    stop
    start
    ;;
  *)
    echo "Usage: $0 {start|stop|restart|uninstall}"
esac
```

上面的脚本需要在最开始添加以下内容才能运行 (等号后面需要添加值)：

```bash
#!/bin/env bash

NAME=
SCRIPT=
RUNAS=
PIDFILE=
LOGFILE=
```

我通过 `Python` 脚本读取 `json` 配置文件，然后将配置文件的内容写入到 `Sys Init V` 的脚本中，
最后将后将这个脚本拷贝到指定目录：

```python
def create_sys_v_init_service(config):
    try:
        with open('script/service_tmp.sh', 'r') as f:
            service_content = f.read()
    except Exception as e:
        command_checker(1, f"Error: {e}")
        return

    header = f'''#!/bin/env bash
NAME={config.serviceName}
SCRIPT="{parse_iterable_into_str([config.serviceStartJavaCommand] +
config.serviceStartJavaArgs + [config.serviceStartJarFile])}"
RUNAS={config.serviceUser}
PIDFILE={config.servicePIDFile}
LOGFILE={config.serviceLogFile}
'''
    service_content = header + service_content
    log_debug(f"service_content:\n {service_content}")

    res = os.system(
        f"echo '{service_content}' | {sudo_cmd} tee {config.serviceSysVInitDirectory}/{config.serviceName}")
    command_checker(res, f"Failed to create {config.serviceSysVInitDirectory}/{config.serviceName}")
    res = os.system(f'{sudo_cmd} chmod +x {config.serviceSysVInitDirectory}/{config.serviceName}')
    command_checker(
        res, f"Failed to chmod +x {config.serviceSysVInitDirectory}/{config.serviceName}")

    if logging.getLogger().level == logging.DEBUG:
        try:
            with open(f'{config.serviceSysVInitDirectory}/{config.serviceName}', 'r') as f:
                log_debug(f"Service content:\n {f.read()}")
        except Exception as e:
            command_checker(1, f"Error: {e}")
            return
```

除了这些更改以外，将依赖的安装交给了 `Python` 脚本管理，`bash` 脚本仅仅负责安装 `python` 依赖。

# Finish the deploy script for database

`pr` 链接：[gcs-pull-25](https://github.com/CMIPT/gcs-back-end/pull/25)

本次的 `pr` 主要完成了数据库部署部分，根据之前提供的 `SQL` 脚本，
我们在部署脚本中调用了 `SQL` 脚本去部署数据库。

在部署数据库部分，先在数据库中检查是否存在指定的用户，不存在就进行创建，
并根据配置文件的密码修改数据库中用户的密码。之后检查是否存在数据库，不存在就创建数据库。
然后给当前用户赋予指定数据库的所有权限。最后就是通过该用户去调用 `SQL` 脚本创建表。

除了数据库部分的部署外，还增加了激活不同配置文件的功能。在 `deploy_helper.py` 中增加了一个函数：

```python
def active_profile(config):
    profile_format = f"spring.profiles.active={parse_iterable_into_str(config.profiles, sep=',')}"
    log_debug(f"Profile format: {profile_format}")
    try:
        lines = None
        if os.path.exists(application_config_file_path):
            with open(application_config_file_path, 'r') as f:
                lines = f.readlines()
        with open(application_config_file_path, 'w') as f:
            if lines:
                for line in lines:
                    if not line.startswith('spring.profiles.active'):
                        f.write(line)
            f.write(profile_format)
    except Exception as e:
        command_checker(1, f"Error: {e}")
```

除此之外，我们将 `Spring Boot` 的相关配置使用 `yml` 格式进行配置，而脚本创建的配置则放置在了
`properties` 文件中。这样能保证后续增加的配置一定能生效，因为 `properties` 文件的优先级高于 `yml`
文件。

# Configure datasource and mybatis-plus

`pr` 链接：[gcs-pull-29](https://github.com/CMIPT/gcs-back-end/pull/29)

在本次的提交中主要完成了数据源的配置和 `mybatis-plus` 的配置。

除此之外，还修复了 [gcs-pull-25](https://github.com/CMIPT/gcs-back-end/pull/25) 中引入的问题：
为 `su` 命令传入了错误的密码。`su` 命令接收的密码应该为操作系统用户的密码，而不是数据库用户的密码，
之前错误地传入了数据库用户的密码。

这个 `pr` 还修复了 `gitaction` 中默认使用的 `Java` 版本为 `11`，通过以下的代码为环境设置 `openjdk-17`：

```yml
- name: Set up openjdk-17
  uses: actions/setup-java@v4
  if: steps.check_java_files.outputs.java_files_exist == 'true'
  with:
    distribution: 'zulu'
    java-version: '17'
```

配置 `druid` 的时候有以下的注意事项：

* 如果是 `Spring Boot 3` 应该使用 `druid-spring-boot-3-starter` 的依赖，而不是 `druid-spring-boot-starter`。
* 配置数据源除了需要添加 `druid` 的依赖外，还需要添加数据驱动的依赖，例如 `postgresql` 驱动应该添加以下依赖：

```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
</dependency>
```

* 除此之外还需要配置 `jdbc` 可以使用 `spring-boot-starter-jdbc` 进行自动装配。
* 引入依赖后需要在 `application.yml` 中配置数据源的相关信息，例如：

```yml
datasource:
    druid:
      username: gcs_debug
      password: gcs_debug
      url: jdbc:postgresql://localhost:5432/gcs_debug
      type: com.alibaba.druid.pool.DruidDataSource
      driver-class-name: org.postgresql.Driver
      initial-size: 5
      min-idle: 5
      max-active: 20
      max-wait: 6000 # unit: ms
      time-between-eviction-runs-millis: 60000
      min-evication-idle-time-millis: 600000 # min alive time of a connection
      max-evication-idle-time-millis: 1200000 # max alive time of a connection
      validation-query: SELECT 1
      test-while-idle: true
      async-init: true
      keep-alive: true
      filters:
        stat:
          enable: true
          log-slow-sql: true
          slow-sql-millis: 1000
        wall:
          enable: true
          log-violation: true
          throw-exception: false
          config:
            drop-table-allow: false
            delete-allow: false
      web-stat-filter:
        enabled: true
        url-pattern: /*
        exclusions: "*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*"
        session-stat-enable: true
        session-stat-max-count: 1000
      stat-view-servlet:
        enabled: true
        url-pattern: /druid/*
        reset-enable: false
        login-username: druid
        login-password: druid
        allow: # empty means allow all
```

# Update the primary key of database

`pr` 链接：[gcs-pull-31](https://github.com/CMIPT/gcs-back-end/pull/31)

在本次提交中，我们主要是修复数据库主键与`MyBatis-Plus` 兼容问题, 以及完善了部分的细节问题。

经过测试发现，`MyBatis-Plus` 与 `PostgreSQL` 数据库在主键的使用上存在一些问题，
由于在数据库中设置了序列`sequence` 和`MyBatis-Plus` 自动增长主键`IdType.AUTO` 类型的功能重复，
导致脚本进行插入的时候`id`会以步长`2`进行自增，所以我们对数据库中的`sequence` 进行了删除。

为了符合阿里巴巴的开发规范，我们还对数据库中的表的主键字段进行了重命名，全部修改为`id`。

由于开发中使用逻辑删除，所以当一个用户进行了注销操作后，真正的数据不会被删除，
当这个用户重新进行注册的时候，由于唯一索引的存在会导致之前的注册邮箱无法使用，
因此目前直接删除了`all_column_constraint.sql`中的唯一索引。

# Finish the example for creating a user

`pr` 链接：[gcs-pull-33](https://github.com/CMIPT/gcs-back-end/pull/33)

在本次提交中，我们简单地完成了一个创建用户的例子。这个例子是一个简单的 `restful api`，接收一个
`json` 格式的请求，然后将请求中的内容存入数据库中。

我们使用 `OpenAPI 3.0` 完成了 `restful api` 的文档，这样能够更好地展示 `restful api` 的使用方法。

同时使用了 `MockMvc` 演示了如何测试这个 `restful api`。

另外，在本次测试的过程中我们发现经过 `MD5` 加密后的密码通常采用 `32` 位的 `16` 进制数表示，
而不是存储 `128` 位的二进制，所以之前的数据库中密码字段的长度设置为 `128` 位是不合适的，
我们将其修改为 `32` 位。

# Add an example for validation

`pr` 链接：[gcs-pull-34](https://github.com/CMIPT/gcs-back-end/pull/34)

在本次的 `pr` 中，我们主要是增加了 `Spring Validation` 如何使用的例子。

从
[使用 `Spring-Validation` 进行参数校验](/blog/2024/spring-validation-intro)
中可以了解到更多关于关于本次提交的信息。

# User record to define the DTO object

`pr` 链接：[gcs-pull-35](https://github.com/CMIPT/gcs-back-end/pull/35)

我们发现对于 `DTO` 和 `VO` 对象，我们在创建之后不再需要对其值进行修改，
因此我们将这两个对象定义为 `record` 对象。

`record` 关键字是 `Java 14` 中引入的，用于定义一个不可变的类。`record` 类似于 `data class`，
它会自动为类的属性生成 `getter` 方法，`equals` 方法，`hashCode` 方法，`toString` 方法。

# Add filter for authentication and authorization

`pr` 链接：[gcs-pull-37](https://github.com/CMIPT/gcs-back-end/pull/37)

这个例子的主要目的是为了实现使用双 `token` 进行认证和授权。关于双 `token` 的使用可以参考
[`JJWT` + `Spring Boot Filter` 实现认证与授权](/blog/2024/filter-jjwt-intro)。

除此之外，我们还通过一些特殊的手段实现了对 `filter` 中抛出的异常进行全局处理，
可以参考[`Spring Boot Filter` 异常处理](/blog/2024/exception-handler-intro)。

最后，在本次的 `pr` 中，我们第一次尝试引入 `DevelopmentController`，
该 `controller` 主要是为前端开发者服务的，
通过请求该 `controller` 可以获取到如所有可用的 `API` 路径信息、所有错误信息等。

# Update the processing of error messages

`pr` 链接：[gcs-pull-38](https://github.com/CMIPT/gcs-back-end/pull/38)

在之前的错误信息处理的过程中，我们只会返回形如以下内容的错误信息：

```json
{
    "message": "Error occurs while converting message"
}
```

这样的错误信息没有错误代码，不适合前端进行比较，因此我们在本次提交中主要增加了一个错误代码枚举类：

```java
package edu.cmipt.gcs.enumeration;

public enum ErrorCodeEnum {
    // This should be ignored, this is to make the ordinal of the enum start from 1
    ZERO_PLACEHOLDER,

    USERDTO_ID_NULL("UserDTO.id.Null"),
    USERDTO_ID_NOTNULL("UserDTO.id.NotNull"),
    USERDTO_USERNAME_SIZE("UserDTO.username.Size"),
    USERDTO_USERNAME_NOTBLANK("UserDTO.username.NotBlank"),
    USERDTO_EMAIL_NOTBLANK("UserDTO.email.NotBlank"),
    USERDTO_EMAIL_EMAIL("UserDTO.email.Email"),
    USERDTO_USERPASSWORD_SIZE("UserDTO.userPassword.Size"),
    USERDTO_USERPASSWORD_NOTBLANK("UserDTO.userPassword.NotBlank"),

    USERSIGNINDTO_USERNAME_NOTBLANK("UserSignInDTO.username.NotBlank"),
    USERSIGNINDTO_USERPASSWORD_NOTBLANK("UserSignInDTO.userPassword.NotBlank"),

    USERNAME_ALREADY_EXISTS("USERNAME_ALREADY_EXISTS"),
    EMAIL_ALREADY_EXISTS("EMAIL_ALREADY_EXISTS"),
    WRONG_SIGN_IN_INFORMATION("WRONG_SIGN_IN_INFORMATION"),

    INVALID_TOKEN("INVALID_TOKEN"),
    ACCESS_DENIED("ACCESS_DENIED"),

    MESSAGE_CONVERSION_ERROR("MESSAGE_CONVERSION_ERROR");

    // code means the error code in the message.properties
    private String code;

    ErrorCodeEnum() {}

    ErrorCodeEnum(String code) {
        this.code = code;
    }

    public String getCode() {
        return code;
    }
}
```

这个枚举类中的 `code` 对应着 `message.properties` 中的错误信息。我们只需要简单实现一个工具类就可以进行
内容的转换。
更多的内容可以查看[`Spring Validation` 与错误代码自定义](/blog/2024/spring-validation-custom-error-code)。

除了上面的修改之外，我们还尽可能避免在代码中出现魔法值，将一些常量提取到了 `Constant` 类中。

最后，在本次的提交中，我们对 `authenticationController` 中的方法添加了测试。

# Add configuration for CORS

`pr` 链接：[gcs-pull-39](https://github.com/CMIPT/gcs-back-end/pull/39)。

在本次的提交中，我们增加了跨域的配置。我们为 `dev` 和 `prod` 两个环境配置了不同的跨域策略。

在 `dev` 中，我们允许所有的请求，而在 `prod` 中，
我们只允许前端发送的 `GET`、`POST` 以及 `DELETE` 请求。
除此之外，我们添加了测试类对 `pord` 环境中的跨与配置进行了测试。

# Add function to get user info by name

`pr` 链接：[gcs-pull-41](https://github.com/CMIPT/gcs-back-end/pull/41)

在这个提交中，我们增加了一个可以通过用户名获取用户完整信息的 `API`。

除此之外，我们将 `Token` 相关的字段全部放入到请求头和相应头中进行返回，而不是在 `body` 中返回。

# Add function for checking the info validity

`pr` 链接：[gcs-pull-42](https://github.com/CMIPT/gcs-back-end/pull/42)

这个提交中，我们增加了用于检查邮箱和用户名合理性的 `API`。这几个 `API` 主要是在用户注册的时候使用，
用于检查用户输入的邮箱和用户名是否合理。
在校验部分，我们通过 `Spring Validation` 对路径变量和请求参数进行了校验，详细的方法可以查看
[`Spring Validation` 路径变量与请求参数校验](/blog/2024/spring-validation-path-variable-and-request-param)

# Add an api for updating user information

`pr` 链接：[gcs-pull-43](https://github.com/CMIPT/gcs-back-end/pull/43)

在本次的提交中，我们不仅增加了更新用户信息的 `API` 还增加了更多对用户信息的校验：

例如通过 `@Pattern` 注解校验用户名和密码的组成字符等。
除此之外我们去除了 `Token` 的请求头，而是通过 `Access-Token` 和 `Refresh-Token` 进行更好的区分。

在更新用户信息的部分，如果涉及用户密码的更改，
我们会将之前的 `Token` 全部通过加入黑名单的方式使其失效并在响应头中添加新的 `Token`。
这也意味着在用户更新的时候必须携带 `Access-Token` 和 `Refresh-Token`。
不过黑名单这部分的内容目前并没有实现 (只是有一个 `stub`)。

对于权限验证部分，我们在 `JwtFilter` 中增加了对更新部分的检验：

我们只允许用户对 `id` 与 `Access-Token` 中 `id` 相同的用户进行更改。

在本次修改的时候我们发现了 `Long` 精度丢失的问题，详细的内容可以查看：
[`Long` 在 `Swagger` 中精度丢失](/blog/2024/long-precision-lost-in-swagger)

在本次修改的时候我们发现了 `request` 中的 `body` 无法被多次读取，详细的内容可以查看：
[`Spring` 多次读取请求体](/blog/2024/spring-read-request-body-multiple-times)

# Add an api for deleting user by id

`pr` 链接：[gcs-pull-45](https://github.com/CMIPT/gcs-back-end/pull/45)。

本次提交增加删除用户的 `API`。在删除用户的时候，我们会将用户的 `Token` 加入到黑名单中，使其失效。

在权限验证部分，我们只允许用户删除自己的账户，
即只有 `id` 与 `Access-Token` 中的 `id` 相同的用户才能删除自己的账户。

# Add an api for getting the repository list by user id

`pr` 链接：[gcs-pull-47](https://github.com/CMIPT/gcs-back-end/pull/47)。

在本次提交中，我们增加了一个通过用户 `id` 获取用户的仓库列表的 `API`。
这是一个分页查询，在这个 `API` 中，我们通过 `@RequestParam` 获取用户的 `id`，
页数和每页的数量。这部分我们使用了 `stream` 将 `UserPO` 对象转换成 `UserVO` 对象：


```java
@GetMapping(ApiPathConstant.USER_PAGE_USER_REPOSITORY_API_PATH)
public List<RepositoryVO> pageUserRepositories(
        @RequestParam("id") Long userId,
        @RequestParam("page") Integer page,
        @RequestParam("size") Integer size,
        @RequestHeader(HeaderParameter.ACCESS_TOKEN) String accessToken) {
    QueryWrapper<RepositoryPO> wrapper = new QueryWrapper<RepositoryPO>();
    String idInToken = JwtUtil.getID(accessToken);
    assert idInToken != null;
    if (!idInToken.equals(userId.toString())) {
        // the user only can see the public repositories of others
        wrapper.eq("is_private", false);
    }
    wrapper.eq("user_id", userId);
    return repositoryService.page(new Page<>(page, size), wrapper).getRecords().stream()
            .map(RepositoryVO::new)
            .collect(Collectors.toList());
}
```

在权限验证部分，当查询他人的仓库时，我们只允许用户查看公开的仓库，而不允许查看私有的仓库。

# Finish the APIs for creating repository

`pr` 链接：[gcs-pull-49](https://github.com/CMIPT/gcs-back-end/pull/49)。

在本次的提交中，我们完成了创建仓库的 `API`。为了实现该 `API`，我们使用
`sudo -u git git init --bare <dir>` 命令来实现以 `git` 用户的身份创建仓库。

除此之外，现在的额外配置会直接追加到 `application.yml` 文件中，而不是进行覆盖写。
因为我们发现后续的相同配置会覆盖之前的配置，
因此我们没有必要先删除之前的配置再添加新的配置，直接追加即可。

在 `Mybatis-Plus` 中有 `list` 接口可以支持分页，
因此我们在分页部分使用 `list` 而不再是 `page` 获取后通过 `getRecords()` 转换成 `List`：

```java
// ...
public List<RepositoryVO> pageUserRepository(...) {
    // ...
    return repositoryService.list(new Page<>(page, size), wrapper).stream()
            .map(RepositoryVO::new)
            .collect(Collectors.toList());
}
```

在本次提交中，我们完成了自定义测试类的执行顺序，可以查看
[`Spring Boot Test` 自定义测试类顺序](/blog/2024/spring-boot-test-custom-test-class-order)
以获取更多信息。

TODO:

- [ ] 完成仓库 `url` 的生成
    - [x] `ssh`
    - [ ] `https`
- [x] 创建仓库部分应启用事务管理

# Finish the APIs related with ssh-key

`pr` 链接：[gcs-pull-51](https://github.com/CMIPT/gcs-back-end/pull/51)。

这个 `pr` 实现了 `ssh` 相关的功能：

* 创建 `ssh-key`
* 获取 `ssh-key` (分页获取)
* 删除 `ssh-key`
* 更新 `ssh-key`

我们为 `ssh` 专门创建了一张 `t_ssh_key` 表用于管理 `ssh-key`。

在这次提交中，我们开启了事务支持，对所有需要额外操作文件系统的操作进行了事务管理 (例如创建仓库)。

我们将 `git` 相关的一些常量移动到了 `GitConstant` 类中，这样能够更好地管理这些常量。

TODO:

- [ ] 修改 `E-R` 图

# Finish delete and update repo, and delete user

`pr` 链接：[gcs-pull-53](https://github.com/CMIPT/gcs-back-end/pull/53)

在这 `pr` 中，我们完成了以下功能：

* 删除仓库
* 更新仓库
* 完善删除用户 (删除用户创建的 `ssh-key`)

对于删除仓库功能，我们会删除仓库的 `git` 文件夹。

对于更新仓库，目前不支持更新仓库的名称。

删除用户部分，我们会同时删除用户创建的 `ssh-key`。

除此之外，我们将所有需要查询数据库才能完成鉴权的操作都放到了 `controller` 层。
