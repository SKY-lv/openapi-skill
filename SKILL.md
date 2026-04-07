---
name: openapi-skill
description: "OpenAPI/Swagger 规范助手。设计、编写、验证 OpenAPI 3.x 规范文档。触发词：openapi、swagger、api文档。"
metadata: {"openclaw": {"emoji": "📚"}}
---

# OpenAPI Skill

## 功能说明

设计、编写和验证 OpenAPI 3.x 规范文档。

## OpenAPI 基础结构

```yaml
openapi: 3.0.3
info:
  title: 我的API
  version: 1.0.0
  description: API描述
servers:
  - url: https://api.example.com/v1
    description: 生产环境
  - url: https://staging.example.com/v1
    description: 测试环境
paths:
  /users:
    get:
      summary: 获取用户列表
      tags: [用户管理]
      parameters:
        - name: page
          in: query
          schema: { type: integer, default: 1 }
        - name: limit
          in: query
          schema: { type: integer, default: 20 }
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
                  total:
                    type: integer
```

## 组件复用

```yaml
components:
  schemas:
    User:
      type: object
      required: [id, email]
      properties:
        id:
          type: string
          format: uuid
        email:
          type: string
          format: email
        name:
          type: string
        avatar:
          type: string
          format: uri
        created_at:
          type: string
          format: date-time
        updated_at:
          type: string
          format: date-time

    Error:
      type: object
      properties:
        code:
          type: string
        message:
          type: string
        details:
          type: object

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

  responses:
    Unauthorized:
      description: 未授权
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
    NotFound:
      description: 资源不存在
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
```

## 安全配置

```yaml
security:
  - bearerAuth: []

paths:
  /admin:
    get:
      security:
        - bearerAuth: []
```

## 认证流程

### 登录获取Token

```yaml
  /auth/login:
    post:
      summary: 用户登录
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [email, password]
              properties:
                email:
                  type: string
                  format: email
                password:
                  type: string
                  format: password
      responses:
        '200':
          description: 登录成功
          content:
            application/json:
              schema:
                type: object
                properties:
                  access_token:
                    type: string
                  refresh_token:
                    type: string
                  expires_in:
                    type: integer
                  user:
                    $ref: '#/components/schemas/User'
```

## 常用HTTP状态码

| 代码 | 含义 |
|------|------|
| 200 | 成功 |
| 201 | 创建成功 |
| 204 | 删除成功，无内容 |
| 400 | 请求参数错误 |
| 401 | 未认证 |
| 403 | 无权限 |
| 404 | 资源不存在 |
| 409 | 资源冲突 |
| 422 | 验证错误 |
| 429 | 请求过于频繁 |
| 500 | 服务器错误 |

## 最佳实践

1. **版本控制**：URL中包含版本号 `/v1/users`
2. **分页**：使用 cursor 或 offset+limit
3. **错误格式**：统一错误响应结构
4. **过滤排序**：Query 参数支持 filter, sort
5. **字段选择**：支持 `fields=id,name,email`

## 验证工具

```bash
# npm
npx @redocly/cli lint openapi.yaml

# Docker
docker run --rm -v $PWD:/spec redocly/cli lint openapi.yaml

# 在线验证
# https://redocly.com/docs/cli/
```

## 常见模式

### CRUD资源

```yaml
/users:
  get:
    summary: 列表
  post:
    summary: 创建

/users/{id}:
  get:
    summary: 获取单个
  put:
    summary: 更新
  delete:
    summary: 删除
```

### 子资源嵌套

```yaml
/users/{userId}/orders:
  get:
    summary: 获取用户的订单
  post:
    summary: 创建订单
```
