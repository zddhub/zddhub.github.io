---
layout: post
title: "使用 Docker-compose 部署 rails 应用到生产环境"
category: Memo
tags: Rails docker deploy docker-compose
---

> 在上一个项目上有两年的工作经验，半年后再次使用 Ruby on Rails，还是掉进了各种坑里。踩过了坑，最重要的就是记录下来。

### 简介

本文主要介绍如何将 Ruby on Rails 应用 docker 化并部署到生产环境。该应用使用 mongodb 数据库，其它数据库方法类似。本文方法不是全自动化部署，但仅需要执行一条命令。

<!-- more -->

### 为什么用 Docker？

选择 docker 的原因有两点：一、容易 Setup 生产环境，在生产环境中安装 [`docker ce`](https://docs.docker.com/install/linux/docker-ce/ubuntu/) 和 [`docker-compose`](https://docs.docker.com/compose/install/) 即可；二、可使用 [`docker cloud`](http://cloud.docker.com/) 自动生成镜像并上传到 [`docker store`](https://store.docker.com/) 上，以供 `docker-compose` 使用。`docker store` 在今年取消了自动部署的功能，这导致了本文方法没能做到全自动化部署，可替换类似服务实现全自动化部署。

### 环境

* ruby version `2.4.1`
* mongo version `4.0.0`
* gem 'rails', `5.1.5`
* gem 'mongoid', `6.1.0`

生产机器上需要安装 [`docker ce`](https://docs.docker.com/install/linux/docker-ce/ubuntu/) 和 [`docker-compose`](https://docs.docker.com/compose/install/)，并登陆 [`docker store`](https://store.docker.com/) 或者 [`docker hub`](https://hub.docker.com/)。

### Ruby on Rails + mongodb 应用

Rails 默认使用 `SQLite` 数据库，用 `ActiveRecord` 实现增删查改。原生支持 `MySQL` 和 `PostgreSQL`，只需要在 `config/database.yml` 里配置即可。如果你想把数据库替换成 `mongodb`，在创建应用(`rails new`)时，需要添加 `--skip-active-record` 参数。如果是已存在的项目，在 `config/application` 里删除 `require 'rails/all'`，仅导入你项目使用的模块，删除 `ActiveRecord`，并删除所有关于 `ActiveRecord` 的配置。我是在踩了所有坑才注意到[官方文档](https://docs.mongodb.com/mongoid/master/tutorials/mongoid-rails/)的。没解决问题前的官方文档就像没人待见的姑娘，静静地躺在那里，别人从跟前经过了千百遍，就是看不见啊！

在 rails `5.1.5` 里，`require 'rails/all'` 被替换成以下代码：

```rb
# config/application.rb
%w(
  action_controller/railtie
  action_view/railtie
  action_mailer/railtie
  active_job/railtie
  action_cable/engine
  rails/test_unit/railtie
  sprockets/railtie
).each do |railtie|
  begin
    require railtie
  rescue LoadError
  end
end
```

### 容器化 Ruby on Rails 应用

本文使用私有项目 [`growth`](https://github.com/zddhub/growth)。

#### dockerfile

基于 ruby 2.4.1 创建自己的 rails 镜像，把 CMD 放在 `docker-compose.yml` 里，方便更改。

```sh
# Dockerfile
FROM ruby:2.4.1
MAINTAINER zddhub <zddhub@gmail.com>

RUN apt-get update && apt-get install -qq -y build-essential nodejs libpq-dev --fix-missing --no-install-recommends

# Set an environment variable to store where the app is installed to inside
# of the Docker image.
ENV INSTALL_PATH /growth
RUN mkdir -p $INSTALL_PATH

# This sets the context of where commands will be ran in and is documented
# on Docker's website extensively.
WORKDIR $INSTALL_PATH

# Ensure gems are cached and only get updated when they change. This will
# drastically increase build times when your gems do not change.
COPY Gemfile Gemfile
RUN bundle install

# Copy in the application code from your work station at the current directory
# over to the working directory.
COPY . .

# Expose a volume so that nginx will be able to read in assets in production.
VOLUME ["$INSTALL_PATH/public"]

# The default command that gets ran will be to start the Unicorn server.
# Run command in `docker-compose.yml`
# CMD bundle exec rails s -b 0.0.0.0
```

#### .dockerignore

在将项目 COPY 到 docker 里时，有一些文件可以不用 COPY，可使用 `.dockerignore` 忽略掉，用法和功能类似 `.gitignore`。

```
# .dockerignore
.git
.dockerignore
Gemfile.lock
deploy
README.md
```

`README.md` 永远是写给人看的，不推荐 COPY 到容器里。

#### 创建自己的镜像

创建完成后，build 自己的镜像：

```sh
docker build -t zddhub/growth .
```

并将镜像 push 到 [`docker store`](https://store.docker.com/) 上。这一步可以自动化，只需要在 [`docker cloud`](http://cloud.docker.com/) 上关联自己的 github 账号，每次 push 后，[`docker cloud`](http://cloud.docker.com/) 都会帮你自动 build 镜像，并 push 到 [`docker store`](https://store.docker.com/) 上。[`docker cloud`](http://cloud.docker.com/) 免费版支持 5 个私有镜像。

#### docker-compose

`docker-compose` 是 docker 官方提供的工具，可以让你容易的配置管理多个 docker container。

```sh
# deploy/docker-compose.yml
version: '2'
services:
  mongo:
    image: mongo:4.0.0
    restart: always
    ports:
      - '27017:27017'
    volumes:
      - /tmp/growth:/data/db
    env_file:
      - .growth.env

  growth:
    image: zddhub/growth:latest
    command: bundle exec rails s -e production
    depends_on:
      - mongo
    volumes:
      - /tmp/growth_log:/growth/log
    ports:
      - '3000:3000'
    env_file:
      - .growth.env
```

docker-compose 使用了创建了两个 services：mongo 和 growth，对外提供服务。docker-compose 直接拉取 build 好的 image `zddhub/growth:latest`，而没有从本地编译，我是不想把代码 copy 到生产环境的。所以单另创建了一个目录 `deploy`，把 docker-compose.yml 文件放在下面，部署时只需要将 `deploy` 目录 copy 到生产环境后执行即可。

#### .growth.env

最值得一提的是 `.growth.env` 文件，保存了服务运行时需要的所有环境变量、密码和密钥等重要信息，只有自己才能看得见。所以请创建后立即加入 `.gitignore` 和 `.dockerfile` 中。

### 运行你的服务

如果一切顺利的话，在 `deploy` 目录下运行：

```
docker-compose up
```

你的服务就启动了，访问 `http://localhost:3000/` 查看效果。如果使用 `CTRL + C` 等方式结束 container，再次 `docker-compose up` 的时候会报错，所以最好每次都 调用下面命令 restart:

```
docker-compose down && docker-compose up
```

### 配置数据库密码

数据库被脱库的案例经常发生，所以更加不能裸奔，设置强壮的数据库密码是必须的。

```rb
# config/mongoid.yml
production:
  # Configure available database clients. (required)
  clients:
    # Defines the default client. (required)
    default:
      # Defines the name of the default database that Mongoid can connect to.
      # (required).
      database: growth_production
      # Provides the hosts the default client can connect to. Must be an array
      # of host:port pairs. (required)
      hosts:
        - mongo:27017
      options:
        user: <%= ENV['MONGO_INITDB_ROOT_USERNAME'] %>
        password: <%= ENV['MONGO_INITDB_ROOT_PASSWORD'] %>
        auth_source: <%= ENV['MONGO_AUTH_SOURCE'] %>
        auth_mech: :scram

  options:
      preload_models: true
```

从环境变量里读取数据库的用户名密码和 `auth_source` 等敏感信息，把环境变量存放在 `.growth.env` 中，切记不要将该文件提交到代码库。在这里不小心把键值 `user` 写成了 `username`，花了大半天才发现，😢。

注意，`.growth.env` 文件格式非常简单，`key=value`, 每个 key/value 一行，不要给 value 加引号，否则引号会做为 value 的一部分：

```
SECRET_KEY_BASE=xxx
RAILS_SERVE_STATIC_FILES=xxx
MONGO_INITDB_ROOT_USERNAME=xxx
MONGO_INITDB_ROOT_PASSWORD=xxx
MONGO_INITDB_DATABASE=xxx
MONGO_AUTH_SOURCE=xxx
```

替换 xxx 后再运行 `docker-compose up` 命令，一切就都准备好了。

### 一键部署到生产环境

虽然 docker cloud 不提供自动部署到生产环境的解决方案了，但是为了生活幸福，写个脚本部署吧：

```sh
#!/bin/bash

# export SERVER='hostname@x.x.x.x'

scp -r deploy $SERVER:~

ssh $SERVER << EOF
    rm -rf deploy
    cd deploy
    docker pull zddhub/growth:latest
    docker-compose down && docker-compose up -d
    cd ..
    rm -rf deploy
    docker image prune -f
    docker container prune -f
EOF

```

在运行前配置你生产环境的服务器地址：`export SERVER='hostname@x.x.x.x'`。运行时先把 `deploy` 目录（包括 `docker-compose.yml` 和 `.growth.env`）copy 到生产环境，部署成功后删除整个目录以防泄密。重要的文件 `.growth.env` 在生产环境上转了一圈，立即被删除，最大可能的降低了风险。还有更便捷安全的方法吗？

### 数据库迁移

在迭代开发中，难免要更新数据库结构，更新老数据相关字段，这就需要对线上数据做 migration。[mongoid_rails_migrations](https://github.com/adacosta/mongoid_rails_migrations) 专门用来干这件事。可惜的是该项目没有人维护，在 Rails 5.0 及以上版本存在一个 [bug](https://github.com/adacosta/mongoid_rails_migrations/pull/44)，都有人修了，但是没人发布一个新版本到  rubygems.org。无奈之下，只能写 task 来修了。创建文件 lib/tasks/migrate.rake，示例如下：

```rb
# lib/tasks/migrate.rake
namespace :migration do
  task hot_fix_2018_06_22: [:environment] do
    # do everything what you want
  end
end
```

在线上运行如下代码：

```sh
docker-compose run growth rake migration:add_session_salt_to_user_2018_06_22 RAILS_ENV=production
```

也可以将其加入到自动化运行脚本中，平时注释起来。如下所示：

```
#!/bin/bash
# ...
ssh $SERVER << EOF
    rm -rf deploy
    cd deploy
    docker pull zddhub/growth:latest
    docker-compose down && docker-compose up -d
    # The below line only to run rake task
    docker-compose run growth rake migration:hot_fix_2018_06_22 RAILS_ENV=production
    cd ~
    rm -rf deploy
    docker image prune -f
    docker container prune -f
EOF
```

这样，不用登陆就可以部署应用，更新数据库啦。

### 良心建议

如果你做了个应用想对外发布的话，请你克制住内心的小骚动，先自己用三天再说。刚上线就挂了的产品，是会让自己和用户都很受伤害的。

### 结论

用 Ruby on Rails 写后台 API 足够简单，快速。借助 docker，很容易通过一键操作部署应用到后台服务器。
