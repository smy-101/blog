---
  title: Rails开发(二)
  pubDate: 2024-11-30
  categories: [后端,Ruby,Rails]
  description: '介绍了Rails开发的基本流程'
---

本篇介绍通过 Rails 实现用户通过密码或验证码注册登录的功能来介绍 Rails 的基础用法

本篇先实现验证码发送相关的功能

## 路由

Rails 的路由配置在 `config/routes.rb` 中，本次我打算采用 restful 风格的路由，在 `routes.rb` 中进行如下配置

```
Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      resource :session, only: [ :create, :destroy ]
      resources :validation_codes, only: [ :create ]
    end
  end
end
```

namespace 代表了一个命名空间，如上代码所示，当请求验证码时请求的路由为 `/api/v1/validation_codes`。

`resource` 的单复数形式表示的是**该种资源是否会有多个**，例如验证码会有多个这里就用了复数的形式，一个用户的会话只有一个所以使用了单数的形式

后面的 `create`, `destroy` 表示路由的动作，only 用于限制生成的路由，默认会生成如下路由

```
GET /profile -> profiles#show
GET /profile/new -> profiles#new
POST /profile -> profiles#create
GET /profile/edit -> profiles#edit
PATCH/PUT /profile -> profiles#update
DELETE /profile -> profiles#destroy
```

可以使用 `rails routes` 命令来查看 rails 项目中的路由情况

## Model

在 Rails 中，模型（Model）是 MVC（模型-视图-控制器）架构中的一部分，负责处理应用程序的业务逻辑和与数据库的交互。模型通常与数据库中的表对应，每个模型类代表数据库中的一张表，每个模型实例代表表中的一行记录。

1. **数据映射**：
    - 模型类与数据库表对应，模型的属性与表的列对应。ActiveRecord 是 Rails 的 ORM（对象关系映射）库，它使得模型类可以直接映射到数据库表，并提供了方便的方法来操作数据库记录。
2. **数据验证**：
    - 模型可以定义数据验证规则，确保数据在保存到数据库之前是有效的。例如，可以验证某个字段是否存在、格式是否正确、值是否唯一等。

`rails generate model ValidationCode` 会生成一个 model 文件

```
class ValidationCode < ApplicationRecord
  validates :email, presence: true # 校验相关
end
```

在生成 model 文件的同时还会生成一个 migration 文件，用于更改数据库，操作在下节数据库中进行了描述
## 数据库

设置完路由后要对数据库进行设置，生成对应的表结构

`rails generate migration CreateValidationCodes`

上述的代码会在 rails 项目中生成一个表的迁移文件，在该文件中填入该表所需的字段和类型

```
class CreateValidationCodes < ActiveRecord::Migration[8.0]
  def change
    create_table :validation_codes do |t|
      t.string :email
      t.integer :kind, default: 1, null: false
      t.string :code, limit: 100
      t.datetime :used_at

      # 自动生成
      t.timestamps
    end
  end
end
```

迁移文件完成之后，使用 `rails db:migrate` 命令将表结构同步到数据库中

### 表结构修改

如果之后要修改原先的表结构，则需要通过命令再创建一个表迁移文件，在新的迁移文件上进行修改操作

`rails generate migration ChangeValidationCodes`

```
class ChangeValidationCodes < ActiveRecord::Migration[8.0]
  def change
    # 表操作
  end
end
```

之后再运行 `rails db:migrate` 即可

## Controller

在 Rails 中，控制器（Controller）是 MVC（模型-视图-控制器）架构中的一部分，负责处理用户请求、与模型交互并选择适当的视图进行渲染。控制器的主要作用是协调模型和视图之间的交互，确保应用程序的业务逻辑得到正确执行。

以下是控制器在 Rails 中的主要作用和功能：
1. **处理请求**：
    - 控制器接收来自客户端的 HTTP 请求，并根据请求的路径和方法（GET、POST、PUT、DELETE 等）调用相应的动作（方法）。
2. **与模型交互**：
    - 控制器从模型中获取数据或将数据保存到模型中。它负责调用模型的方法来执行业务逻辑和数据操作。
3. **选择视图**：
    - 控制器选择适当的视图模板来渲染响应。视图模板通常是 HTML 文件，包含嵌入的 Ruby 代码（ERB）来显示数据。
4. **处理参数**：
    - 控制器从请求中提取参数，并将这些参数传递给模型或用于其他逻辑处理。
5. **重定向和渲染**：
    - 控制器可以重定向到另一个动作或 URL，或者渲染特定的视图模板。重定向通常用于处理表单提交后的导航，而渲染用于生成响应内容。
6. **过滤器**：
    - 控制器可以使用过滤器（before_action、after_action、around_action）在动作执行前后执行特定的代码。过滤器通常用于认证、授权、日志记录等。

```
rails generate controller ValidationCodes
```

```
class Api::V1::ValidationCodesController < ApplicationController
  def create
    validation_code = ValidationCode.new email: params[:email], kind: :sign_in
    if validation_code.save
      render status: 200
    else
      render json: { errors: validation_code.errors }, status: 400
    end
  end
end
```

## 测试

由于我使用的 Rails 是 api 模式所以没有 View 相关的东西，然后为了提供 API 文档给外部，我才用了 Rspec + Rswag 的方案，通过单元测试的方法来生成一个通用的 swagger 文档

```
# Gemfile

gem "rswag"

group :development, :test do
  gem "rspec-rails", "~> 7.0.0"
end
```

在 Gemfile 中添加需要的 gem 包后, `bundle install` 安装

```
rails generate rspec:install

rails generate rswag:install
```

由于使用的是 api 模式大部分的测试都是测接口

```
rails generate rspec:request api/v1/validation_codes
```

```
require 'swagger_helper'

RSpec.describe 'ValidationCodes', type: :request do
  path '/api/v1/validation_codes' do
    post '发送验证码' do
      tags 'ValidationCodes'
      consumes 'application/json'
      parameter name: :validation_code, in: :body, schema: {
        type: :object,
        properties: {
          email: { type: :string }
        },
        required: [ 'email' ]
      }

      response '200', '验证码发送成功' do
        let(:validation_code) { { email: 'shimy1011@foxmail.com' } }

        run_test! do |response|
          expect(response).to have_http_status(:ok)
        end
      end

      response '429', '请求过于频繁' do
        let(:validation_code) { { email: 'shimy1011@foxmail.com' } }

        before do
          # 先发送一次请求，期望返回200
          post '/api/v1/validation_codes', params: validation_code.to_json, headers: { 'Content-Type': 'application/json' }
          expect(response).to have_http_status(:ok)
        end

        run_test! do |response|
          # 第二次请求，期望返回429
          post '/api/v1/validation_codes', params: validation_code.to_json, headers: { 'Content-Type': 'application/json' }
          expect(response).to have_http_status(:too_many_requests)
        end
      end
    end
  end
end
```

最后通过命令查看单元测试是否通过:

```
bundle exec rspec
```

## 总结

大致的流程如下:

1. 约定路由
2. 生成 model
3. 根据 model 生成表结构，同步到数据库
4. 生成 controller，生成测试文件，提前写出接口的要求
5. 根据测试文件完成 controller
6. 接口通过测试

这里只粗略的讲解下 rails 开发的一个基本流程，并没有太详细的将如何做。这个主要是在 Rails 中有比较多的约定的内容，光靠一次文字描述基本上讲不清。

不过只要理清思路，Rails 的开发流程还是相当顺畅的，剩下的就是多实践
