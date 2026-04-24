---
title: Tool集成指南
date: 2026-04-18
tags:
  - skills
  - 工具集成
  - MCP
  - API集成
  - 函数调用
  - 人工智能
categories:
  - 人工智能/人工智能工具实操/Skills书写与整合/Skills工具集成
alias: [Tool集成, 工具集成指南, Tool Integration]
---

## 关键词

| 集成类型 | 核心概念 | 技术要点 |
|---------|----------|---------|
| MCP集成 | Model Context Protocol | 标准化接口 |
| API集成 | REST/GraphQL/gRPC | 认证、限流 |
| 函数调用 | Function Calling | 参数定义 |
| 外部服务 | Webhook/Event | 异步交互 |
| 权限控制 | RBAC/ABAC | 最小权限 |
| 工具注册 | 注册表、发现机制 | 元数据管理 |

---

# Tool集成概述

## 1.1 Tool在Skills系统中的角色

Tools（工具）是Skills系统与外部世界交互的桥梁。没有Tools，Skills只能处理静态文本；有了Tools，Skills可以读写文件、执行命令、调用API、访问数据库，从而成为真正有用的AI应用。

Tools的集成质量直接影响Skills的能力边界。一个设计良好的Tool集成方案应该具备以下特性：

**可发现性**：Tools可以被Skills方便地发现和引用。

**可组合性**：多个Tools可以协同工作，组合成更复杂的功能。

**可测试性**：每个Tool都可以独立测试和验证。

**可扩展性**：新增Tool不需要修改现有代码。

**安全性**：Tool的调用受到严格的权限控制。

## 1.2 Tool集成的技术架构

```
┌─────────────────────────────────────────────────────────────┐
│                        Skills Layer                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ Skill A     │  │ Skill B     │  │ Skill C     │         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘         │
└─────────┼────────────────┼────────────────┼────────────────┘
          │                │                │
          └────────────────┼────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                     Tool Registry                            │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Tool Discovery │ Permission │ Rate Limiting │ Audit │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│   File System   │ │   Network API   │ │   Databases     │
└─────────────────┘ └─────────────────┘ └─────────────────┘
```

---

# MCP工具集成

## 2.1 MCP协议概述

MCP（Model Context Protocol）是一种开放标准协议，用于在AI系统和外部工具之间建立标准化的通信接口。MCP的设计目标是让AI应用能够轻松地集成各种外部工具和服务。

MCP的核心组件：

- **MCP Server**：提供工具和服务的主机
- **MCP Client**：连接到Server的客户端
- **MCP Resources**：Server提供的数据资源
- **MCP Tools**：Server提供的可调用工具
- **MCP Prompts**：预定义的提示模板

## 2.2 在Skills中声明MCP工具

```yaml
---
name: file-manipulation-skill
version: 1.0.0
description: 文件操作技能集
tools:
  # 内置MCP工具
  - type: mcp
    name: filesystem_read
    server: mcp-filesystem
    description: 读取文件内容
    parameters:
      path:
        type: string
        required: true
        description: 要读取的文件路径
      encoding:
        type: string
        default: utf-8
        description: 文件编码
        
  - type: mcp
    name: filesystem_write
    server: mcp-filesystem
    description: 写入文件内容
    parameters:
      path:
        type: string
        required: true
      content:
        type: string
        required: true
      create_parents:
        type: boolean
        default: false
---

# 文件操作技能

## 支持的操作
1. 读取文件内容
2. 写入文件内容
3. 创建目录
4. 列出目录内容
5. 删除文件/目录

## 使用限制
- 单次读取最大10MB
- 单次写入最大5MB
- 不支持二进制文件直接操作
```

## 2.3 MCP工具的权限配置

```yaml
mcp_permissions:
  server: mcp-filesystem
  
  allowed_operations:
    - read
    - write
    - list
    
  denied_operations:
    - delete
    - execute
    
  path_restrictions:
    allowed_prefixes:
      - /Users/project/src
      - /Users/project/tests
    denied_prefixes:
      - /Users/project/secrets
      - /Users/project/.env
      
  rate_limits:
    requests_per_minute: 60
    burst_size: 10
```

## 2.4 MCP工具的响应处理

```yaml
tool_response_handling:
  success:
    format: structured
    fields: [result, metadata, timestamp]
    
  error:
    format: structured
    fields: [error_code, error_message, details]
    retry_on_codes:
      - network_error
      - timeout
      - service_unavailable
      
  timeout:
    default: 30000
    max: 60000
    on_timeout: fail_with_message
```

> [!tip]
> MCP工具的响应应该包含足够的上下文信息，便于Skills层进行后续处理和调试。

---

# API调用集成

## 3.1 REST API集成模式

在Skills中集成REST API需要定义清晰的结构：

```yaml
---
name: weather-query-skill
version: 1.0.0
description: 天气查询技能
tools:
  - name: query_weather
    type: http_api
    config:
      base_url: https://api.weather.example.com
      method: GET
      endpoint: /v1/current
      headers:
        Authorization: "Bearer {api_key}"
        Content-Type: application/json
      query_params:
        city: "{city_name}"
        units: "{temperature_unit}"
    response_mapping:
      temperature: "$.current.temp"
      humidity: "$.current.humidity"
      description: "$.current.description"
    error_handling:
      retry:
        max_attempts: 3
        backoff: exponential
      fallback:
        on_failure: return_unknown
---

# 天气查询技能

## 使用方法
用户输入城市名称，返回当前天气信息。

## API配置
- 基础URL: https://api.weather.example.com
- 认证方式: Bearer Token
- 响应格式: JSON

## 错误处理
1. 网络错误：最多重试3次，指数退避
2. API错误：根据状态码返回友好的错误信息
3. 限流：自动等待后重试
```

## 3.2 GraphQL集成

```yaml
tools:
  - name: query_graphql
    type: graphql
    config:
      endpoint: https://api.example.com/graphql
      headers:
        Authorization: "Bearer {token}"
      query: |
        query GetUser($id: ID!) {
          user(id: $id) {
            id
            name
            email
            posts {
              id
              title
            }
          }
        }
      variables:
        id: "{user_id}"
    response_mapping:
      user_id: "$.data.user.id"
      user_name: "$.data.user.name"
      post_count: "count($.data.user.posts)"
```

## 3.3 API认证管理

```yaml
authentication:
  type: bearer_token
  config:
    token_source: env
    env_var: WEATHER_API_KEY
    refresh:
      enabled: false
      refresh_endpoint: ""
      
# 或者OAuth2
authentication:
  type: oauth2
  config:
    client_id: "{client_id}"
    client_secret: "{client_secret}"
    token_endpoint: https://auth.example.com/oauth/token
    scopes:
      - read:user
      - read:data
    token_storage: secure
    refresh:
      enabled: true
      before_expiry: 300  # 提前5分钟刷新
```

---

# 函数调用（Function Calling）

## 4.1 Function Calling概述

Function Calling是让AI模型主动调用预定义函数的能力。在Skills系统中，这通常通过声明函数的签名和让AI决定何时调用来实现。

## 4.2 函数定义格式

```yaml
functions:
  - name: calculate
    description: 执行数学计算
    parameters:
      type: object
      properties:
        expression:
          type: string
          description: 数学表达式，如 "2 + 3 * 4"
        precision:
          type: integer
          description: 小数精度，默认2
          default: 2
      required: ["expression"]
    returns:
      type: object
      properties:
        result:
          type: number
          description: 计算结果
        expression:
          type: string
          description: 原始表达式
    examples:
      - input:
          expression: "10 / 3"
          precision: 4
        output:
          result: 3.3333
          expression: "10 / 3"
```

## 4.3 函数执行流程

```yaml
function_execution:
  lifecycle:
    - name: declaration
      description: Skill声明可调用的函数
      
    - name: invocation
      description: AI决定调用哪个函数
      
    - name: validation
      description: 验证参数是否有效
      
    - name: execution
      description: 执行函数逻辑
      
    - name: response
      description: 返回执行结果给AI
      
    - name: integration
      description: AI整合结果生成最终响应
      
  validation:
    strict: true
    type_check: true
    allowed_values: true
    custom_rules:
      - name: expression_safety
        description: 防止危险表达式
        check: no_system_commands
```

## 4.4 函数调用安全

```yaml
function_security:
  # 沙箱执行
  sandbox:
    enabled: true
    isolation_level: process
    
  # 代码执行限制
  code_execution:
    allowed_languages: []
    blocked_patterns:
      - __import__
      - eval
      - exec
      - os.system
      
  # 资源限制
  resource_limits:
    max_execution_time: 5000  # 毫秒
    max_memory_mb: 128
    max_output_size: 1048576  # 1MB
```

---

# 外部服务集成

## 5.1 Webhook集成

Webhook允许外部系统主动向Skills系统推送数据：

```yaml
webhook:
  endpoint: /webhook/github
  methods: [POST]
  authentication:
    type: signature
    header: X-Hub-Signature-256
    secret: "{webhook_secret}"
  events:
    - push
    - pull_request
    - issue
  payload_transform:
    github_event: "{event_type}"
    repository: "{repository.full_name}"
    action: "{action}"
```

## 5.2 事件驱动集成

```yaml
event_system:
  type: pubsub
  config:
    broker: redis
    channel_prefix: skills_events
    
  subscriptions:
    - event: user.message
      handler: SKILL.message-handler
      
    - event: data.updated
      handler: SKILL.data-sync
      
    - event: alert.triggered
      handler: SKILL.alert-manager
      
  event_format:
    schema_version: "1.0"
    required_fields: [event_id, event_type, timestamp, data]
```

## 5.3 异步任务处理

```yaml
async_tasks:
  queue: redis
  config:
    host: localhost
    port: 6379
    db: 0
    
  task_types:
    - name: long_running_analysis
      handler: SKILL.analysis-executor
      timeout: 300000  # 5分钟
      retry: 3
      
    - name: batch_processing
      handler: SKILL.batch-processor
      timeout: 600000  # 10分钟
      retry: 1
```

---

# 工具注册与管理

## 6.1 工具注册表

```yaml
tool_registry:
  # 工具元数据存储
  tools:
    - id: tool-001
      name: file_reader
      category: filesystem
      version: 1.2.0
      owner: platform-team
      status: active
      capabilities:
        - read_text
        - read_binary
      metadata:
        created: 2026-01-01
        updated: 2026-04-01
        usage_count: 15420
        
    - id: tool-002
      name: web_search
      category: network
      version: 2.1.0
      owner: search-team
      status: active
      capabilities:
        - search
        - fetch
```

## 6.2 工具发现机制

```yaml
tool_discovery:
  methods:
    - type: registry
      description: 从中央注册表发现
      
    - type: file_scan
      description: 扫描指定目录
      paths:
        - ./skills/tools
        - ./shared/tools
        
    - type: mcp_discovery
      description: 从MCP服务器发现
      
  search:
    by_name: true
    by_category: true
    by_tag: true
    by_capability: true
```

## 6.3 工具版本管理

```yaml
version_management:
  strategy: semantic
  deprecation:
    notice_period: 30d
    graceful_shutdown: true
    migration_assistance: true
  compatibility:
    check_on_load: true
    warn_on_version_mismatch: true
```

---

# 工具权限控制

## 7.1 权限模型概述

权限控制确保Tools只被授权的Skills和场景使用：

```yaml
permission_model:
  type: rbac  # Role-Based Access Control
  roles:
    - name: admin
      permissions: ["*"]  # 所有权限
      
    - name: developer
      permissions:
        - filesystem:read
        - filesystem:write
        - api:read
        
    - name: viewer
      permissions:
        - filesystem:read
        
    - name: restricted
      permissions:
        - api:read
```

## 7.2 细粒度权限配置

```yaml
permissions:
  skill: SKILL.code-reviewer
  
  filesystem:
    read:
      allowed: true
      paths:
        - /Users/project/src
        - /Users/project/tests
    write:
      allowed: false
    delete:
      allowed: false
      exception:
        - pattern: "*.tmp"
          allowed: true
          
  network:
    allowed_hosts:
      - api.github.com
      - api.weather.com
    blocked_hosts:
      - internal.corp.local
    rate_limit: 100/minute
```

## 7.3 审计日志

```yaml
audit:
  enabled: true
  level: detailed
  
  logged_events:
    - tool_invocation
    - permission_denied
    - rate_limit_exceeded
    - authentication_failure
    
  log_format:
    timestamp: ISO8601
    skill_id: string
    tool_name: string
    parameters: object
    result: object
    duration_ms: integer
    user: string
    
  retention:
    days: 90
    storage: secure
```

> [!warning]
> 权限控制应该遵循最小权限原则——只授予完成任务所需的最小权限集。过度的权限可能导致安全风险。

---

# 相关文档

- [[Skills基础理论]] - Skills核心概念
- [[SKILL_md语法详解]] - SKILL.md格式
- [[Skills设计模式]] - 设计模式
- [[Skills测试与优化]] - 测试策略
- [[Skills调试实战]] - 调试方法

---

*本文档详细介绍了在Skills系统中集成各类工具的方法和最佳实践。*
