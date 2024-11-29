---
  title: Rails开发(一)
  pubDate: 2024-11-29
  categories: [后端,Ruby,Rails]
  description: '介绍了如何使用Rails创建一个基础项目'
---

最近学习了 Ruby，打算使用 Ruby on Rails 进行一次后端开发，了解下开发流程

本篇用于介绍如何使用 Ruby on Rails 创建一个基础项目

因为 Ruby 官方对 windows 的开发支持几乎没有，所以一般来说开发 Ruby 项目使用的都是 MacOS 或者是 Linux 系统。本次开发项目选用的是 Windows 的 WSL 环境来进行开发

## WSL

WSL 我选取的是 Arch 发行版，安装 WSL 请另行搜索官方文档

## 创建项目

这次我准备使用 RoR 来提供后端服务，使用 postgresql 来作为数据库，开发时数据库放到 WSL 中的 docker 容器中。docker 的安装请自行搜索网上资料。

Ruby 使用 rbenv 来提供版本控制

```
sudo pacman -S --needed base-devel rust libffi libyaml openssl zlib

yay -S rbenv ruby-build

rbenv install 3.3.6

rbenv global 3.3.6

# 切换ruby-china源 可不切换
gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/

gem install bundler

# 切换ruby-china源 可不切换
bundle config set --global mirror.https://rubygems.org https://gems.ruby-china.com

gem install rails
```

```
# postgresql相关依赖
yay -S postgresql-libs

rails new --api --database=postgresql --skip-test <name>
```

如上代码，就可创建一个 Rails 项目，之后还需要在 Rails 项目中配置数据库相关配置

## 启动项目

为了更好的管理数据库，我将数据库配置写成了 docker compose 文件，放到了项目中

```
# docker=compose.dev.yml
version: '3.8'

services:
  db:
    image: postgres:16
    container_name: <name>_postgres  # 给容器一个固定名称，便于引用
    restart: unless-stopped
    ports:
      - "5432:5432"  # 映射到本机端口，使本机Ruby环境可以访问
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: <name>_development
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres-data:
    name: <name>_postgres_data  # 给数据卷一个固定名称
```

```
# 启动开发数据库
# 没有安装过docker 自行安装
docker compose -f docker-compose.test.yml up -d
```

之后在 `config/database.yml` 中进行配置

```
development:
  <<: *default

  #在本地使用
  database: stashbox_development
  host: localhost
  port: 5432
  username: postgres
  password: postgres
```

配置完后即可启动项目:

```
rails s
```

成功运行后，即可在 `http:127.0.0.1:3000` 页面看到 rails 的默认页面，此时基础配置完成

## 另一种配置方案

开发 RoR 也可以在通过 VSCode 的 Dev Container 插件在 docker 容器中进行开发。这种开发方案的好处是可以将容器的配置保存到项目中，不仅可以在本地进行开发，还可以使用 Github 的 CodeSpace 来进行远程开发

需要在项目中新建一个 `.devcontainer` 的文件夹，放入配置文件

```
# .devcontainer/devcontainer.json
{
  "name": "Rails Development",
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/workspace",
  // 使用新的customizations格式
  "customizations": {
    "vscode": {
      // VS Code 设置
      "settings": {
        "terminal.integrated.defaultProfile.linux": "bash",
        "editor.tabSize": 2,
        "files.trimTrailingWhitespace": true,
        "editor.formatOnSave": true,
        "files.trimFinalNewlines": true,
        "files.insertFinalNewline": true,
        "editor.rulers": [
          80,
          120
        ],
        // Ruby 相关设置
        "ruby.useBundler": true,
        "ruby.useLanguageServer": true,
        "ruby.intellisense": "rubyLocate",
        "ruby.format": "rubocop",
        "ruby.lint": {
          "rubocop": {
            "useBundler": true
          }
        },
        "ruby.interpreter.path": "/usr/local/bin/ruby",
        "rubyLsp.rubyVersionManager": {
          "type": "none"
        }
      },
      // 推荐的扩展
      "extensions": [
        "Shopify.ruby-lsp",
        "miguel-savignano.ruby-symbols",
        "Hridoy.rails-snippets",
        "groksrc.ruby",
        "misogi.ruby-rubocop",
        "kaiwood.endwise"
      ]
    }
  },
  // 容器创建后运行的命令
  "postCreateCommand": "bundle install",
  // 转发的端口
  "forwardPorts": [
    3000,
    5432
  ],
  // 其他特性标记
  "features": {
    "ghcr.io/devcontainers/features/git:1": {}
  }
}
```

```
# .devcontainer/docker-compose.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        - HTTP_PROXY=${HTTP_PROXY}
        - HTTPS_PROXY=${HTTPS_PROXY}
        - NO_PROXY=${NO_PROXY}
    volumes:
      - ..:/workspace:cached
      - bundle:/usr/local/bundle
      - rails_cache:/workspace/tmp/cache
    command: sleep infinity
    environment:
      - POSTGRES_HOST=db
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - RAILS_ENV=development
      - HTTP_PROXY=${HTTP_PROXY}
      - HTTPS_PROXY=${HTTPS_PROXY}
      - http_proxy=${HTTP_PROXY}
      - https_proxy=${HTTPS_PROXY}
      - NO_PROXY=localhost,127.0.0.1,db,${NO_PROXY}
      - no_proxy=localhost,127.0.0.1,db,${NO_PROXY}
    depends_on:
      - db

  db:
    image: postgres:16
    restart: unless-stopped
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: <name>_development

volumes:
  postgres-data:
  bundle:
  rails_cache:
```

```
# .devcontainer/Dockerfile
FROM ruby:3.3.6

# 安装常用工具和依赖
RUN apt-get update -qq && \
    apt-get install -y build-essential libpq-dev nodejs npm && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# 安装 Rails 依赖
RUN gem install bundler

# 设置工作目录
WORKDIR /workspace

# 设置PATH
ENV PATH="/workspace/bin:${PATH}"

# 设置默认的Rails环境
ENV RAILS_ENV=development
```

由于对 Rails 开发没有经验，担心后期修改环境配置比较麻烦，就暂时先在本地进行开发
