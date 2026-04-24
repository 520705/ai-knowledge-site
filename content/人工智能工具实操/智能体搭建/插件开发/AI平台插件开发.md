---
title: AI平台插件开发：从入门到精通
date: 2026-04-24
tags:
  - 插件开发
  - Dify
  - n8n节点
  - Coze插件
  - OpenAI兼容
  - SDK开发
  - 认证集成
categories:
  - 智能体搭建
  - 插件开发
alias: AI Plugin Development
description: 深入讲解Dify插件、n8n自定义节点、Coze插件SDK的开发方法，包含完整的认证集成、工具定义、API对接实战，以及OpenAI兼容格式的统一开发框架。
---

# AI平台插件开发：从入门到精通

> [!NOTE] 这篇指南讲什么
> 插件是扩展AI平台能力的核心方式。这篇指南从原理到实战，详细讲解Dify插件、n8n自定义节点、Coze插件SDK的开发方法，包含完整的认证集成、API对接、工具定义实战，帮助你构建可复用的AI能力扩展。

## 核心关键词速览

| 关键词 | 说明 | 关键词 | 说明 |
|--------|------|--------|------|
| Dify插件 | Dify扩展能力 | n8n自定义节点 | 节点开发 |
| Coze SDK | Coze插件开发 | OpenAI格式 | 兼容开发 |
| API导入 | 快速集成 | OAuth认证 | 安全认证 |
| 工具定义 | Tool Schema | 认证管理 | 凭证安全 |
| 节点注册 | Node Registry | 凭证加密 | 安全存储 |

## 1. 插件开发概述

### 1.1 为什么需要插件？

你有没有遇到过这种情况：

- 想用某个API，但平台没有现成的集成
- 需要连接私有服务，但官方不支持
- 想要自定义的处理逻辑，但平台的功能不够灵活

**插件就是来解决这些问题的。**

打个比方：如果AI平台是一台手机，插件就是App Store里的各种应用。你可以根据自己的需求，安装各种插件来扩展平台的能力。

### 1.2 插件架构对比

| 平台 | 插件类型 | 开发语言 | 扩展方式 | 学习曲线 |
|------|----------|----------|----------|----------|
| **Dify** | 工具/模型/插件包 | Python | 代码+配置 | 中等 |
| **n8n** | 自定义节点 | TypeScript | npm包 | 较陡 |
| **Coze** | 插件/机器人 | API规范 | OpenAPI导入 | 简单 |
| **LangChain** | 工具/工具包 | Python/JS | 装饰器 | 简单 |

### 1.3 插件核心要素

```
┌─────────────────────────────────────────────────────────────┐
│                        插件架构图                                  │
│                                                            │
│   ┌─────────────────────────────────────────────┐          │
│   │              插件包 (Plugin Package)          │          │
│   │                                              │          │
│   │  ┌──────────┐  ┌──────────┐  ┌──────────┐  │          │
│   │  │ 元数据    │  │ 认证模块  │  │ 工具定义  │  │          │
│   │  │ manifest │  │ credentials│ │ tool.yaml│  │          │
│   │  └──────────┘  └──────────┘  └──────────┘  │          │
│   │                                              │          │
│   │  ┌──────────────────────────────────────┐  │          │
│   │  │              业务逻辑                   │          │
│   │  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ │          │
│   │  │  │ API调用  │ │ 数据处理 │ │ 结果转换 │ │          │
│   │  │  └─────────┘ └─────────┘ └─────────┘ │          │
│   │  └──────────────────────────────────────┘  │          │
│   └─────────────────────────────────────────────┘          │
└─────────────────────────────────────────────────────────────┘
```

### 1.4 什么时候需要开发插件？

| 场景 | 推荐方式 | 说明 |
|------|----------|------|
| 使用标准API | 直接集成 | 大多数API都有现成插件 |
| 连接私有服务 | 自定义插件 | 企业内部系统、无公开API |
| 特殊处理逻辑 | 自定义节点 | 需要复杂的数据转换 |
| 批量集成 | SDK开发 | 需要程序化调用多个API |

## 2. Dify插件开发

### 2.1 Dify插件架构

Dify的插件系统非常灵活，可以扩展：
- **工具插件**：给Agent提供外部能力
- **模型插件**：支持新的AI模型
- **预测插件**：自定义推理逻辑

```
dify_plugin/
├── __init__.py                 # 包初始化
├── manifest.yaml               # 插件元数据
├── credentials.py              # 认证配置
├── tools/                      # 工具目录
│   ├── __init__.py
│   ├── weather_tool.py         # 天气工具
│   ├── search_tool.py          # 搜索工具
│   └── tool.yaml               # 工具定义
├── models/                     # 模型目录（可选）
│   ├── __init__.py
│   └── custom_model.py
└── README.md                   # 说明文档
```

### 2.2 完整插件开发实战

#### 第一步：创建插件元数据

```yaml
# manifest.yaml
identifier: "enterprise-services-plugin"
name: "企业服务插件"
description: |
  提供企业常用的服务集成，包括：
  - 员工信息查询
  - 任务管理系统
  - 审批流程
  - 文档管理
  
  适用于企业内部AI助手场景。
version: "1.0.0"
author: "Enterprise Team <team@example.com>"
homepage: "https://github.com/example/dify-enterprise-plugin"
icon: "./icon.png"

# 插件类型
type: "custom"  # official/community/custom

# 支持的Dify版本
dify_version: ">=0.6.0"

# 依赖包
dependencies:
  requests: "^2.31.0"
  aiohttp: "^3.9.0"
  pydantic: "^2.5.0"

# 凭证配置
credentials:
  - name: "api_key"
    label: "API密钥"
    type: "secret-input"
    required: true
    placeholder: "请输入您的API密钥"
    help_text: "在企业服务后台获取API密钥"
    
  - name: "base_url"
    label: "服务地址"
    type: "text-input"
    required: true
    default: "https://api.enterprise.com"
    placeholder: "https://api.enterprise.com"
    help_text: "企业API服务的基础地址"

  - name: "timeout"
    label: "超时时间（秒）"
    type: "number-input"
    required: false
    default: 30
    min: 5
    max: 120
```

#### 第二步：实现认证管理

```python
# credentials.py
from typing import Optional, Dict, Any
from pydantic import BaseModel, Field, validator
from dify_plugin import ToolCredential


class EnterpriseCredentials(BaseModel):
    """企业服务认证配置"""
    api_key: str = Field(..., description="API密钥")
    base_url: str = Field(default="https://api.enterprise.com", description="服务地址")
    timeout: int = Field(default=30, ge=5, le=120, description="超时时间（秒）")
    
    @validator('api_key')
    def validate_api_key(cls, v):
        if not v or len(v) < 10:
            raise ValueError("API密钥格式不正确")
        return v
    
    @validator('base_url')
    def validate_base_url(cls, v):
        if not v.startswith(('http://', 'https://')):
            raise ValueError("服务地址必须是有效的URL")
        return v.rstrip('/')
    
    def headers(self) -> Dict[str, str]:
        """生成认证请求头"""
        return {
            "Authorization": f"Bearer {self.api_key}",
            "Content-Type": "application/json",
            "Accept": "application/json",
            "User-Agent": "Dify-Plugin/1.0"
        }
    
    def to_curl(self) -> str:
        """生成curl命令示例"""
        return f"""
curl -X GET "{self.base_url}/api/v1/ping" \\
  -H "Authorization: Bearer {self.api_key[:8]}..." \\
  -H "Content-Type: application/json"
"""


class CredentialManager:
    """凭证管理器"""
    
    def __init__(self):
        self._cache: Dict[str, EnterpriseCredentials] = {}
    
    @classmethod
    def from_credentials_dict(
        cls, 
        credentials: Dict[str, Any]
    ) -> EnterpriseCredentials:
        """从凭证字典创建配置对象"""
        return EnterpriseCredentials(
            api_key=credentials.get('api_key', ''),
            base_url=credentials.get('base_url', 'https://api.enterprise.com'),
            timeout=credentials.get('timeout', 30)
        )
    
    def get_cached(
        self, 
        credential_id: str,
        credentials: Dict[str, Any]
    ) -> EnterpriseCredentials:
        """获取缓存的凭证"""
        if credential_id not in self._cache:
            self._cache[credential_id] = self.from_credentials_dict(credentials)
        return self._cache[credential_id]
    
    def invalidate(self, credential_id: str):
        """使缓存失效"""
        self._cache.pop(credential_id, None)


# 全局凭证管理器实例
credential_manager = CredentialManager()
```

#### 第三步：实现工具类

```python
# tools/employee_tool.py
import requests
from typing import Dict, Any, List, Optional, Type
from dify_plugin import Tool, ToolService, ToolParameter
from pydantic import BaseModel, Field


class EmployeeQueryInput(BaseModel):
    """员工查询输入参数"""
    employee_id: Optional[str] = Field(None, description="员工ID")
    department: Optional[str] = Field(None, description="部门名称")
    name: Optional[str] = Field(None, description="员工姓名（模糊搜索）")
    limit: int = Field(default=10, ge=1, le=100, description="返回数量")


class Employee:
    """员工数据结构"""
    def __init__(
        self,
        employee_id: str,
        name: str,
        department: str,
        email: str,
        position: str,
        phone: str = None,
        hire_date: str = None
    ):
        self.employee_id = employee_id
        self.name = name
        self.department = department
        self.email = email
        self.position = position
        self.phone = phone
        self.hire_date = hire_date
    
    def to_dict(self) -> Dict[str, Any]:
        return {
            "员工ID": self.employee_id,
            "姓名": self.name,
            "部门": self.department,
            "职位": self.position,
            "邮箱": self.email,
            "电话": self.phone or "未登记",
            "入职日期": self.hire_date or "未知"
        }


class EmployeeTool(Tool):
    """员工查询工具"""
    
    # ========== 工具元数据 ==========
    name = "query_employee"
    version = "1.0.0"
    description = """
    查询企业员工信息。

    支持三种查询方式：
    1. 按员工ID精确查询：返回单个员工详细信息
    2. 按部门查询：返回部门下所有员工列表
    3. 按姓名搜索：返回姓名匹配的员工列表

    返回结果包含员工的联系方式、职位、部门等信息。
    """
    
    # ========== 参数定义 ==========
    parameters = [
        {
            "name": "query_type",
            "type": "select",
            "label": "查询类型",
            "description": "选择查询方式",
            "required": True,
            "options": [
                {"value": "by_id", "label": "按员工ID查询"},
                {"value": "by_department", "label": "按部门查询"},
                {"value": "by_name", "label": "按姓名搜索"}
            ]
        },
        {
            "name": "employee_id",
            "type": "string",
            "label": "员工ID",
            "description": "当查询类型为「按员工ID查询」时必填",
            "required": False
        },
        {
            "name": "department",
            "type": "string",
            "label": "部门名称",
            "description": "输入部门名称，如「技术部」「销售部」",
            "required": False
        },
        {
            "name": "name",
            "type": "string",
            "label": "员工姓名",
            "description": "输入员工姓名，支持模糊匹配",
            "required": False
        },
        {
            "name": "limit",
            "type": "number",
            "label": "返回数量",
            "description": "最多返回的员工数量",
            "required": False,
            "default": 10,
            "min": 1,
            "max": 100
        }
    ]
    
    # ========== 工具实现 ==========
    
    def invoke(self, tool_parameters: Dict[str, Any]) -> Dict[str, Any]:
        """执行员工查询"""
        from credentials import credential_manager
        
        # 获取凭证
        credentials = credential_manager.get_cached(
            self.runtime.credentials_id,
            self.runtime.credentials
        )
        
        # 解析参数
        query_type = tool_parameters.get('query_type', 'by_id')
        
        try:
            if query_type == 'by_id':
                return self._query_by_id(
                    tool_parameters.get('employee_id'),
                    credentials
                )
            elif query_type == 'by_department':
                return self._query_by_department(
                    tool_parameters.get('department'),
                    tool_parameters.get('limit', 10),
                    credentials
                )
            elif query_type == 'by_name':
                return self._query_by_name(
                    tool_parameters.get('name'),
                    tool_parameters.get('limit', 10),
                    credentials
                )
            else:
                return {
                    "success": False,
                    "error": f"不支持的查询类型: {query_type}"
                }
                
        except requests.Timeout:
            return {
                "success": False,
                "error": f"请求超时，请稍后重试"
            }
        except requests.RequestException as e:
            return {
                "success": False,
                "error": f"API调用失败: {str(e)}"
            }
    
    def _query_by_id(
        self,
        employee_id: str,
        credentials
    ) -> Dict[str, Any]:
        """按ID查询"""
        if not employee_id:
            return {
                "success": False,
                "error": "员工ID不能为空"
            }
        
        response = requests.get(
            f"{credentials.base_url}/api/v1/employees/{employee_id}",
            headers=credentials.headers(),
            timeout=credentials.timeout
        )
        response.raise_for_status()
        
        data = response.json()
        employee = Employee(**data)
        
        return {
            "success": True,
            "data": employee.to_dict(),
            "query_type": "by_id"
        }
    
    def _query_by_department(
        self,
        department: str,
        limit: int,
        credentials
    ) -> Dict[str, Any]:
        """按部门查询"""
        if not department:
            return {
                "success": False,
                "error": "部门名称不能为空"
            }
        
        response = requests.get(
            f"{credentials.base_url}/api/v1/employees",
            params={"department": department, "limit": limit},
            headers=credentials.headers(),
            timeout=credentials.timeout
        )
        response.raise_for_status()
        
        data = response.json()
        employees = [Employee(**emp).to_dict() for emp in data.get('employees', [])]
        
        return {
            "success": True,
            "data": employees,
            "total": len(employees),
            "department": department,
            "query_type": "by_department"
        }
    
    def _query_by_name(
        self,
        name: str,
        limit: int,
        credentials
    ) -> Dict[str, Any]:
        """按姓名搜索"""
        if not name:
            return {
                "success": False,
                "error": "员工姓名不能为空"
            }
        
        response = requests.get(
            f"{credentials.base_url}/api/v1/employees/search",
            params={"name": name, "limit": limit},
            headers=credentials.headers(),
            timeout=credentials.timeout
        )
        response.raise_for_status()
        
        data = response.json()
        employees = [Employee(**emp).to_dict() for emp in data.get('employees', [])]
        
        return {
            "success": True,
            "data": employees,
            "total": len(employees),
            "keyword": name,
            "query_type": "by_name"
        }
    
    def validate_credentials(self, credentials: Dict[str, Any]) -> bool:
        """验证凭证有效性"""
        try:
            creds = credential_manager.from_credentials_dict(credentials)
            # 尝试调用验证接口
            response = requests.get(
                f"{creds.base_url}/api/v1/ping",
                headers=creds.headers(),
                timeout=5
            )
            return response.status_code == 200
        except Exception:
            return False
```

#### 第四步：注册工具

```python
# tools/__init__.py
from .employee_tool import EmployeeTool
from .task_tool import TaskTool
from .document_tool import DocumentTool

# 导出所有工具
TOOLS = [
    EmployeeTool,
    TaskTool,
    DocumentTool,
]

__all__ = ['TOOLS', 'EmployeeTool', 'TaskTool', 'DocumentTool']
```

### 2.3 工具定义YAML（可选）

如果不想用Python定义，也可以用YAML定义工具：

```yaml
# tools/tool.yaml
schema_version: "1.0"

tools:
  - name: query_employee
    label: "查询员工"
    description: "查询企业员工信息，支持按ID、部门、姓名查询"
    icon: "user-search"
    
    parameters:
      - name: query_type
        type: select
        required: true
        options:
          - value: by_id
            label: "按员工ID"
          - value: by_department
            label: "按部门"
          - value: by_name
            label: "按姓名"
      
      - name: employee_id
        type: string
        required: false
        label: "员工ID"
        show_when: query_type == "by_id"
      
      - name: department
        type: string
        required: false
        label: "部门名称"
        show_when: query_type == "by_department"
      
      - name: name
        type: string
        required: false
        label: "员工姓名"
        show_when: query_type == "by_name"
      
      - name: limit
        type: number
        required: false
        default: 10
        min: 1
        max: 100
        label: "返回数量"

  - name: create_task
    label: "创建任务"
    description: "在任务管理系统中创建新任务"
    
    parameters:
      - name: title
        type: string
        required: true
        label: "任务标题"
        max_length: 200
      
      - name: description
        type: text
        required: false
        label: "任务描述"
        max_length: 2000
      
      - name: assignee_id
        type: string
        required: false
        label: "负责人ID"
      
      - name: due_date
        type: datetime
        required: false
        label: "截止日期"
      
      - name: priority
        type: select
        required: false
        default: normal
        options:
          - value: low
            label: "低"
          - value: normal
            label: "普通"
          - value: high
            label: "高"
          - value: urgent
            label: "紧急"
```

### 2.4 插件安装与调试

```bash
# 1. 打包插件
cd dify_plugin
zip -r enterprise-plugin.zip . -x "*.pyc" -x "__pycache__/*"

# 2. 在Dify中安装
# Dify控制台 -> 设置 -> 插件 -> 上传插件 -> 选择zip文件

# 3. 配置凭证
# 插件管理 -> 企业服务插件 -> 添加凭证 -> 填写API密钥

# 4. 测试工具
# 在聊天中测试: "查询员工ID为001的员工信息"
```

## 3. n8n自定义节点开发

### 3.1 n8n节点架构

n8n的节点开发比Dify更接近原生开发，需要：
- TypeScript代码
- npm包管理
- 编译打包

```
n8n-nodes-enterprise/
├── package.json              # 包配置
├── tsconfig.json            # TypeScript配置
├── src/
│   ├── nodes/
│   │   └── Enterprise/       # 节点目录
│   │       ├── Enterprise.node.ts       # 节点主文件
│   │       ├── Enterprise.node.description.ts  # 节点描述
│   │       └── icons/                    # 图标
│   └── credentials/
│       └── EnterpriseApi.credentials.ts  # 凭证定义
├── dist/                    # 编译输出
├── README.md
└── SPEC.md                  # 规范文档
```

### 3.2 完整节点开发实战

#### 第一步：初始化项目

```bash
# 创建项目
mkdir n8n-nodes-enterprise && cd n8n-nodes-enterprise
npm init -y

# 安装依赖
npm install n8n-core n8n-workflow typescript @types/node

# 开发依赖
npm install -D typescript ts-node @typescript-eslint/parser
```

```json
// package.json
{
  "name": "n8n-nodes-enterprise",
  "version": "1.0.0",
  "description": "企业服务n8n节点集合",
  "keywords": ["n8n", "nodes", "enterprise"],
  "license": "MIT",
  "main": "dist/Enterprise.node.js",
  "scripts": {
    "build": "tsc",
    "dev": "ts-node src/nodes/Enterprise/Enterprise.node.ts",
    "lint": "eslint src/**/*.ts"
  },
  "n8n": {
    "nodes": ["dist/Enterprise.node.js"]
  }
}
```

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

#### 第二步：定义凭证

```typescript
// src/credentials/EnterpriseApi.credentials.ts
import {
  ICredentialType,
  INodeProperties,
} from 'n8n-workflow';

export class EnterpriseApiCredentials implements ICredentialType {
  name = 'enterpriseApi';
  displayName = '企业API凭证';
  documentationUrl = 'https://docs.example.com/api';
  
  properties: INodeProperties[] = [
    {
      displayName: 'API密钥',
      name: 'apiKey',
      type: 'string',
      typeOptions: {
        password: true,
      },
      default: '',
      placeholder: '请输入API密钥',
      description: '在企业服务后台获取的API密钥',
    },
    {
      displayName: '服务地址',
      name: 'baseUrl',
      type: 'string',
      default: 'https://api.enterprise.com',
      placeholder: 'https://api.enterprise.com',
      description: '企业API服务的基础URL地址',
    },
    {
      displayName: '超时时间（毫秒）',
      name: 'timeout',
      type: 'number',
      typeOptions: {
        minValue: 1000,
        maxValue: 120000,
      },
      default: 30000,
      description: '请求超时时间，默认30000毫秒',
    },
  ];
}
```

#### 第三步：定义节点描述

```typescript
// src/nodes/Enterprise/Enterprise.node.description.ts
import type { INodeTypeDescription } from 'n8n-workflow';

export const enterpriseNodeDescription: INodeTypeDescription = {
  displayName: '企业服务',
  name: 'enterprise',
  icon: 'file:icons/enterprise.svg',
  group: ['transform'],
  version: 1,
  description: '与企业服务系统集成，提供员工查询、任务管理等功能',
  defaults: {
    name: '企业服务',
  },
  
  // 输入：可以是多个
  inputs: ['main'],
  
  // 输出：可以是多个
  outputs: ['main'],
  
  // 凭证要求
  credentials: [
    {
      name: 'enterpriseApi',
      required: true,
    },
  ],
  
  // 节点属性
  properties: [
    // ===== 基本设置 =====
    {
      displayName: '操作',
      name: 'operation',
      type: 'options',
      noDataExpression: false,
      options: [
        {
          name: '查询员工',
          value: 'queryEmployee',
          description: '查询企业员工信息',
          action: '查询员工',
        },
        {
          name: '创建任务',
          value: 'createTask',
          description: '创建新任务',
          action: '创建任务',
        },
        {
          name: '查询任务',
          value: 'queryTask',
          description: '查询任务信息',
          action: '查询任务',
        },
        {
          name: '更新任务',
          value: 'updateTask',
          description: '更新任务状态',
          action: '更新任务',
        },
        {
          name: '发送通知',
          value: 'sendNotification',
          description: '发送通知消息',
          action: '发送通知',
        },
      ],
      default: 'queryEmployee',
      displayOptions: {
        show: {
          resource: ['employee'],
        },
      },
    },
    
    // ===== 资源选择 =====
    {
      displayName: '资源',
      name: 'resource',
      type: 'options',
      noDataExpression: false,
      options: [
        {
          name: '员工',
          value: 'employee',
        },
        {
          name: '任务',
          value: 'task',
        },
        {
          name: '通知',
          value: 'notification',
        },
      ],
      default: 'employee',
    },
    
    // ===== 员工查询参数 =====
    {
      displayName: '查询类型',
      name: 'queryType',
      type: 'options',
      displayOptions: {
        show: {
          resource: ['employee'],
          operation: ['queryEmployee'],
        },
      },
      options: [
        {
          name: '按ID查询',
          value: 'byId',
        },
        {
          name: '按部门查询',
          value: 'byDepartment',
        },
        {
          name: '搜索',
          value: 'search',
        },
      ],
      default: 'byId',
    },
    
    {
      displayName: '员工ID',
      name: 'employeeId',
      type: 'string',
      required: true,
      displayOptions: {
        show: {
          resource: ['employee'],
          operation: ['queryEmployee'],
          queryType: ['byId'],
        },
      },
      default: '',
      description: '要查询的员工ID',
    },
    
    {
      displayName: '部门名称',
      name: 'department',
      type: 'string',
      required: true,
      displayOptions: {
        show: {
          resource: ['employee'],
          operation: ['queryEmployee'],
          queryType: ['byDepartment'],
        },
      },
      default: '',
      description: '要查询的部门名称',
    },
    
    {
      displayName: '搜索关键词',
      name: 'searchKeyword',
      type: 'string',
      required: true,
      displayOptions: {
        show: {
          resource: ['employee'],
          operation: ['queryEmployee'],
          queryType: ['search'],
        },
      },
      default: '',
      description: '搜索关键词（姓名、工号等）',
    },
    
    {
      displayName: '返回数量',
      name: 'limit',
      type: 'number',
      typeOptions: {
        minValue: 1,
        maxValue: 100,
      },
      displayOptions: {
        show: {
          resource: ['employee'],
          operation: ['queryEmployee'],
          queryType: ['byDepartment', 'search'],
        },
      },
      default: 10,
      description: '最多返回的记录数',
    },
    
    // ===== 任务参数 =====
    {
      displayName: '任务标题',
      name: 'taskTitle',
      type: 'string',
      required: true,
      displayOptions: {
        show: {
          resource: ['task'],
          operation: ['createTask'],
        },
      },
      default: '',
      placeholder: '输入任务标题',
    },
    
    {
      displayName: '任务描述',
      name: 'taskDescription',
      type: 'string',
      typeOptions: {
        rows: 4,
      },
      displayOptions: {
        show: {
          resource: ['task'],
          operation: ['createTask'],
        },
      },
      default: '',
      placeholder: '输入任务描述',
    },
    
    {
      displayName: '负责人',
      name: 'assignee',
      type: 'string',
      displayOptions: {
        show: {
          resource: ['task'],
          operation: ['createTask'],
        },
      },
      default: '',
      placeholder: '负责人ID或邮箱',
    },
    
    {
      displayName: '截止日期',
      name: 'dueDate',
      type: 'dateTime',
      displayOptions: {
        show: {
          resource: ['task'],
          operation: ['createTask'],
        },
      },
      default: '',
      placeholder: '选择截止日期',
    },
    
    {
      displayName: '优先级',
      name: 'priority',
      type: 'options',
      displayOptions: {
        show: {
          resource: ['task'],
          operation: ['createTask'],
        },
      },
      options: [
        { name: '低', value: 'low' },
        { name: '普通', value: 'normal' },
        { name: '高', value: 'high' },
        { name: '紧急', value: 'urgent' },
      ],
      default: 'normal',
    },
    
    // ===== 任务ID（更新/查询用）=====
    {
      displayName: '任务ID',
      name: 'taskId',
      type: 'string',
      required: true,
      displayOptions: {
        show: {
          resource: ['task'],
          operation: ['queryTask', 'updateTask'],
        },
      },
      default: '',
    },
    
    {
      displayName: '新状态',
      name: 'taskStatus',
      type: 'options',
      displayOptions: {
        show: {
          resource: ['task'],
          operation: ['updateTask'],
        },
      },
      options: [
        { name: '待处理', value: 'pending' },
        { name: '进行中', value: 'in_progress' },
        { name: '已完成', value: 'completed' },
        { name: '已取消', value: 'cancelled' },
      ],
      default: 'in_progress',
    },
    
    // ===== 通知参数 =====
    {
      displayName: '接收人',
      name: 'recipient',
      type: 'string',
      required: true,
      displayOptions: {
        show: {
          resource: ['notification'],
          operation: ['sendNotification'],
        },
      },
      default: '',
      placeholder: '接收人ID或邮箱',
    },
    
    {
      displayName: '通知内容',
      name: 'message',
      type: 'string',
      typeOptions: {
        rows: 4,
      },
      required: true,
      displayOptions: {
        show: {
          resource: ['notification'],
          operation: ['sendNotification'],
        },
      },
      default: '',
      placeholder: '输入通知内容',
    },
    
    {
      displayName: '通知类型',
      name: 'notificationType',
      type: 'options',
      displayOptions: {
        show: {
          resource: ['notification'],
          operation: ['sendNotification'],
        },
      },
      options: [
        { name: '站内信', value: 'internal' },
        { name: '邮件', value: 'email' },
        { name: '短信', value: 'sms' },
        { name: '全部', value: 'all' },
      ],
      default: 'internal',
    },
  ],
};
```

#### 第四步：实现节点逻辑

```typescript
// src/nodes/Enterprise/Enterprise.node.ts
import {
  IExecuteFunctions,
  INodeExecutionData,
  INodeType,
  INodeTypeDescription,
  IWebhookData,
  ILoadOptionsFunctions,
  NodeApiError,
  NodeOperationError,
} from 'n8n-workflow';

import { enterpriseNodeDescription } from './Enterprise.node.description';
import { EnterpriseApiCredentials } from '../../credentials/EnterpriseApi.credentials';

export class Enterprise implements INodeType {
  description: INodeTypeDescription = enterpriseNodeDescription;

  // 凭证类型
  credentials: INodeTypeDescription['credentials'] = [
    {
      name: 'enterpriseApi',
      required: true,
    },
  ];

  async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
    // 获取所有输入数据
    const items = this.getInputData();
    const returnData: INodeExecutionData[] = [];

    // 获取凭证
    const credentials = await this.getCredentials('enterpriseApi') as {
      apiKey: string;
      baseUrl: string;
      timeout: number;
    };

    // 获取资源类型和操作
    const resource = this.getNodeParameter('resource', 0) as string;
    const operation = this.getNodeParameter('operation', 0) as string;

    for (let i = 0; i < items.length; i++) {
      try {
        let responseData;

        if (resource === 'employee') {
          responseData = await handleEmployeeOperation(
            this,
            credentials,
            operation,
            i
          );
        } else if (resource === 'task') {
          responseData = await handleTaskOperation(
            this,
            credentials,
            operation,
            i
          );
        } else if (resource === 'notification') {
          responseData = await handleNotificationOperation(
            this,
            credentials,
            operation,
            i
          );
        }

        // 添加到返回数据
        if (Array.isArray(responseData)) {
          returnData.push(...responseData.map(item => ({ json: item })));
        } else {
          returnData.push({ json: responseData });
        }

      } catch (error) {
        // 错误处理
        if (this.continueOnFail()) {
          returnData.push({
            json: {
              error: error.message,
            },
          });
          continue;
        }
        throw new NodeApiError(this.getNode(), error);
      }
    }

    return [returnData];
  }
}

// ============== 员工操作处理 ==============

async function handleEmployeeOperation(
  context: IExecuteFunctions,
  credentials: { apiKey: string; baseUrl: string; timeout: number },
  operation: string,
  itemIndex: number
) {
  const queryType = context.getNodeParameter('queryType', itemIndex) as string;

  switch (queryType) {
    case 'byId': {
      const employeeId = context.getNodeParameter('employeeId', itemIndex) as string;
      return await queryEmployeeById(credentials, employeeId);
    }

    case 'byDepartment': {
      const department = context.getNodeParameter('department', itemIndex) as string;
      const limit = context.getNodeParameter('limit', itemIndex) as number;
      return await queryEmployeesByDepartment(credentials, department, limit);
    }

    case 'search': {
      const keyword = context.getNodeParameter('searchKeyword', itemIndex) as string;
      const limit = context.getNodeParameter('limit', itemIndex) as number;
      return await searchEmployees(credentials, keyword, limit);
    }

    default:
      throw new NodeOperationError(
        context.getNode(),
        `Unknown query type: ${queryType}`
      );
  }
}

async function queryEmployeeById(
  credentials: { apiKey: string; baseUrl: string },
  employeeId: string
) {
  const response = await fetch(
    `${credentials.baseUrl}/api/v1/employees/${employeeId}`,
    {
      headers: {
        'Authorization': `Bearer ${credentials.apiKey}`,
        'Content-Type': 'application/json',
      },
    }
  );

  if (!response.ok) {
    throw new Error(`Failed to query employee: ${response.statusText}`);
  }

  return await response.json();
}

async function queryEmployeesByDepartment(
  credentials: { apiKey: string; baseUrl: string },
  department: string,
  limit: number
) {
  const url = new URL(`${credentials.baseUrl}/api/v1/employees`);
  url.searchParams.set('department', department);
  url.searchParams.set('limit', limit.toString());

  const response = await fetch(url.toString(), {
    headers: {
      'Authorization': `Bearer ${credentials.apiKey}`,
      'Content-Type': 'application/json',
    },
  });

  if (!response.ok) {
    throw new Error(`Failed to query employees: ${response.statusText}`);
  }

  const data = await response.json();
  return data.employees || [];
}

async function searchEmployees(
  credentials: { apiKey: string; baseUrl: string },
  keyword: string,
  limit: number
) {
  const url = new URL(`${credentials.baseUrl}/api/v1/employees/search`);
  url.searchParams.set('q', keyword);
  url.searchParams.set('limit', limit.toString());

  const response = await fetch(url.toString(), {
    headers: {
      'Authorization': `Bearer ${credentials.apiKey}`,
      'Content-Type': 'application/json',
    },
  });

  if (!response.ok) {
    throw new Error(`Failed to search employees: ${response.statusText}`);
  }

  const data = await response.json();
  return data.employees || [];
}

// ============== 任务操作处理 ==============

async function handleTaskOperation(
  context: IExecuteFunctions,
  credentials: { apiKey: string; baseUrl: string },
  operation: string,
  itemIndex: number
) {
  switch (operation) {
    case 'createTask':
      return await createTask(context, credentials, itemIndex);
    case 'queryTask':
      return await queryTask(context, credentials, itemIndex);
    case 'updateTask':
      return await updateTask(context, credentials, itemIndex);
    default:
      throw new NodeOperationError(
        context.getNode(),
        `Unknown task operation: ${operation}`
      );
  }
}

async function createTask(
  context: IExecuteFunctions,
  credentials: { apiKey: string; baseUrl: string },
  itemIndex: number
) {
  const taskData = {
    title: context.getNodeParameter('taskTitle', itemIndex) as string,
    description: context.getNodeParameter('taskDescription', itemIndex) as string,
    assignee: context.getNodeParameter('assignee', itemIndex) as string || undefined,
    dueDate: context.getNodeParameter('dueDate', itemIndex) as string || undefined,
    priority: context.getNodeParameter('priority', itemIndex) as string,
  };

  const response = await fetch(`${credentials.baseUrl}/api/v1/tasks`, {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${credentials.apiKey}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(taskData),
  });

  if (!response.ok) {
    throw new Error(`Failed to create task: ${response.statusText}`);
  }

  return await response.json();
}

async function queryTask(
  context: IExecuteFunctions,
  credentials: { apiKey: string; baseUrl: string },
  itemIndex: number
) {
  const taskId = context.getNodeParameter('taskId', itemIndex) as string;

  const response = await fetch(
    `${credentials.baseUrl}/api/v1/tasks/${taskId}`,
    {
      headers: {
        'Authorization': `Bearer ${credentials.apiKey}`,
        'Content-Type': 'application/json',
      },
    }
  );

  if (!response.ok) {
    throw new Error(`Failed to query task: ${response.statusText}`);
  }

  return await response.json();
}

async function updateTask(
  context: IExecuteFunctions,
  credentials: { apiKey: string; baseUrl: string },
  itemIndex: number
) {
  const taskId = context.getNodeParameter('taskId', itemIndex) as string;
  const status = context.getNodeParameter('taskStatus', itemIndex) as string;

  const response = await fetch(
    `${credentials.baseUrl}/api/v1/tasks/${taskId}`,
    {
      method: 'PATCH',
      headers: {
        'Authorization': `Bearer ${credentials.apiKey}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ status }),
    }
  );

  if (!response.ok) {
    throw new Error(`Failed to update task: ${response.statusText}`);
  }

  return await response.json();
}

// ============== 通知操作处理 ==============

async function handleNotificationOperation(
  context: IExecuteFunctions,
  credentials: { apiKey: string; baseUrl: string },
  operation: string,
  itemIndex: number
) {
  if (operation === 'sendNotification') {
    const recipient = context.getNodeParameter('recipient', itemIndex) as string;
    const message = context.getNodeParameter('message', itemIndex) as string;
    const notificationType = context.getNodeParameter(
      'notificationType',
      itemIndex
    ) as string;

    const response = await fetch(`${credentials.baseUrl}/api/v1/notifications`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${credentials.apiKey}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        recipient,
        message,
        type: notificationType,
      }),
    });

    if (!response.ok) {
      throw new Error(`Failed to send notification: ${response.statusText}`);
    }

    return await response.json();
  }

  throw new NodeOperationError(
    context.getNode(),
    `Unknown notification operation: ${operation}`
  );
}
```

### 3.3 节点打包与安装

```bash
# 1. 编译TypeScript
npm run build

# 2. 链接到n8n开发目录
cd ~/.n8n
npm link /path/to/n8n-nodes-enterprise

# 或者直接复制到节点目录
cp -r dist/* ~/.n8n/nodes/

# 3. 重启n8n
# 访问 http://localhost:5678 找到新节点
```

## 4. Coze插件开发

### 4.1 Coze插件类型

Coze的插件开发有三种方式：

| 类型 | 说明 | 适用场景 |
|------|------|----------|
| **官方插件** | Coze提供的现成插件 | 常用服务 |
| **API导入** | 通过OpenAPI规范导入 | 已有REST API |
| **自定义插件** | 使用SDK开发 | 复杂逻辑 |

### 4.2 通过OpenAPI导入插件

这是最简单的方式，只需要准备好API规范：

```yaml
# openapi.yaml - OpenAPI 3.0规范
openapi: 3.0.3
info:
  title: 企业服务API
  description: |
    企业服务系统API，提供员工管理、任务管理、审批流程等功能。
    
    使用前需要配置API密钥认证。
  version: 1.0.0
  contact:
    email: api@example.com

servers:
  - url: https://api.enterprise.com
    description: 生产环境
  - url: https://api-sandbox.enterprise.com
    description: 沙箱环境

security:
  - BearerAuth: []

components:
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: API Key
  
  schemas:
    Employee:
      type: object
      properties:
        employee_id:
          type: string
          description: 员工ID
        name:
          type: string
          description: 姓名
        department:
          type: string
          description: 部门
        email:
          type: string
          format: email
          description: 邮箱
        position:
          type: string
          description: 职位
        
    Task:
      type: object
      properties:
        task_id:
          type: string
          description: 任务ID
        title:
          type: string
          description: 任务标题
        description:
          type: string
          description: 任务描述
        status:
          type: string
          enum: [pending, in_progress, completed, cancelled]
          description: 任务状态
        priority:
          type: string
          enum: [low, normal, high, urgent]
          description: 优先级
        assignee:
          type: string
          description: 负责人ID
        due_date:
          type: string
          format: date-time
          description: 截止日期

paths:
  # ===== 员工相关 =====
  /api/v1/employees/{employee_id}:
    get:
      operationId: getEmployee
      summary: 获取员工信息
      description: 根据员工ID获取详细信息
      tags:
        - 员工管理
      security:
        - BearerAuth: []
      parameters:
        - name: employee_id
          in: path
          required: true
          schema:
            type: string
          description: 员工ID
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Employee'
        '404':
          description: 员工不存在
          
  /api/v1/employees:
    get:
      operationId: listEmployees
      summary: 获取员工列表
      description: 根据条件获取员工列表
      tags:
        - 员工管理
      security:
        - BearerAuth: []
      parameters:
        - name: department
          in: query
          schema:
            type: string
          description: 部门名称
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
          description: 返回数量
        - name: offset
          in: query
          schema:
            type: integer
            default: 0
          description: 偏移量
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                type: object
                properties:
                  employees:
                    type: array
                    items:
                      $ref: '#/components/schemas/Employee'
                  total:
                    type: integer
                    
  # ===== 任务相关 =====
  /api/v1/tasks:
    post:
      operationId: createTask
      summary: 创建任务
      description: 创建新的任务
      tags:
        - 任务管理
      security:
        - BearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required:
                - title
              properties:
                title:
                  type: string
                  description: 任务标题
                description:
                  type: string
                  description: 任务描述
                assignee:
                  type: string
                  description: 负责人ID
                due_date:
                  type: string
                  format: date-time
                  description: 截止日期
                priority:
                  type: string
                  enum: [low, normal, high, urgent]
                  default: normal
      responses:
        '201':
          description: 创建成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Task'
                
    get:
      operationId: listTasks
      summary: 获取任务列表
      description: 获取任务列表
      tags:
        - 任务管理
      security:
        - BearerAuth: []
      parameters:
        - name: status
          in: query
          schema:
            type: string
            enum: [pending, in_progress, completed, cancelled]
          description: 任务状态
        - name: assignee
          in: query
          schema:
            type: string
          description: 负责人ID
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                type: object
                properties:
                  tasks:
                    type: array
                    items:
                      $ref: '#/components/schemas/Task'
                      
  /api/v1/tasks/{task_id}:
    get:
      operationId: getTask
      summary: 获取任务详情
      tags:
        - 任务管理
      parameters:
        - name: task_id
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Task'
                
    patch:
      operationId: updateTask
      summary: 更新任务
      description: 更新任务信息
      tags:
        - 任务管理
      parameters:
        - name: task_id
          in: path
          required: true
          schema:
            type: string
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                status:
                  type: string
                title:
                  type: string
                description:
                  type: string
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Task'
```

### 4.3 Coze插件导入步骤

```
1. 登录Coze国际版 (https://www.coze.com)
2. 进入工作空间 → 插件 → 创建插件
3. 选择「从API导入」
4. 上传OpenAPI规范文件（YAML/JSON）
5. 配置认证信息：
   - Auth Type: Bearer Token
   - Token Location: Header
   - Header Name: Authorization
   - Token Prefix: Bearer
6. 配置变量：
   - baseUrl: https://api.enterprise.com
7. 测试连接
8. 保存并发布
```

### 4.4 Coze JavaScript SDK开发

对于更复杂的插件，可以使用SDK开发：

```javascript
// coze-plugin-sdk/my-plugin/index.js
const { CozePlugin, Tool, ToolParameter } = require('@coze/sdk');

class EnterprisePlugin extends CozePlugin {
  constructor() {
    super({
      name: 'enterprise',
      version: '1.0.0',
      description: '企业服务插件',
      author: 'Enterprise Team',
    });

    // 注册工具
    this.registerTools();
  }

  registerTools() {
    // 注册员工查询工具
    this.registerTool(
      'query_employee',
      {
        name: 'query_employee',
        description: '查询企业员工信息',
        parameters: [
          new ToolParameter({
            name: 'query_type',
            type: 'string',
            enum: ['by_id', 'by_department', 'search'],
            required: true,
            description: '查询类型',
          }),
          new ToolParameter({
            name: 'employee_id',
            type: 'string',
            required: false,
            description: '员工ID（按ID查询时必填）',
          }),
          new ToolParameter({
            name: 'department',
            type: 'string',
            required: false,
            description: '部门名称（按部门查询时必填）',
          }),
          new ToolParameter({
            name: 'keyword',
            type: 'string',
            required: false,
            description: '搜索关键词（搜索时必填）',
          }),
          new ToolParameter({
            name: 'limit',
            type: 'number',
            required: false,
            default: 10,
            description: '返回数量',
          }),
        ],
      },
      this.queryEmployee.bind(this)
    );

    // 注册任务管理工具
    this.registerTool(
      'manage_task',
      {
        name: 'manage_task',
        description: '创建或更新任务',
        parameters: [
          new ToolParameter({
            name: 'action',
            type: 'string',
            enum: ['create', 'update', 'query'],
            required: true,
            description: '操作类型',
          }),
          new ToolParameter({
            name: 'task_id',
            type: 'string',
            required: false,
            description: '任务ID（更新和查询时必填）',
          }),
          new ToolParameter({
            name: 'title',
            type: 'string',
            required: false,
            description: '任务标题（创建和更新时填写）',
          }),
          new ToolParameter({
            name: 'description',
            type: 'string',
            required: false,
            description: '任务描述',
          }),
          new ToolParameter({
            name: 'status',
            type: 'string',
            enum: ['pending', 'in_progress', 'completed', 'cancelled'],
            required: false,
            description: '任务状态',
          }),
          new ToolParameter({
            name: 'assignee',
            type: 'string',
            required: false,
            description: '负责人ID',
          }),
          new ToolParameter({
            name: 'due_date',
            type: 'string',
            required: false,
            description: '截止日期（ISO格式）',
          }),
          new ToolParameter({
            name: 'priority',
            type: 'string',
            enum: ['low', 'normal', 'high', 'urgent'],
            required: false,
            default: 'normal',
            description: '优先级',
          }),
        ],
      },
      this.manageTask.bind(this)
    );
  }

  // 工具实现
  async queryEmployee(params) {
    const { query_type, employee_id, department, keyword, limit = 10 } = params;
    
    const credentials = await this.getCredentials();
    const { apiKey, baseUrl } = credentials;

    let url, method, body;

    if (query_type === 'by_id') {
      if (!employee_id) {
        throw new Error('员工ID不能为空');
      }
      url = `${baseUrl}/api/v1/employees/${employee_id}`;
      method = 'GET';
    } else if (query_type === 'by_department') {
      if (!department) {
        throw new Error('部门名称不能为空');
      }
      url = `${baseUrl}/api/v1/employees`;
      method = 'GET';
    } else if (query_type === 'search') {
      if (!keyword) {
        throw new Error('搜索关键词不能为空');
      }
      url = `${baseUrl}/api/v1/employees/search`;
      method = 'GET';
    }

    // 构建请求
    const response = await fetch(url, {
      method,
      headers: {
        'Authorization': `Bearer ${apiKey}`,
        'Content-Type': 'application/json',
      },
      // GET请求的参数处理
      ...(method === 'GET' ? {
        url: new URL(url).toString() + `?${new URLSearchParams({ department, q: keyword, limit })}`
      } : {}),
    });

    if (!response.ok) {
      throw new Error(`API请求失败: ${response.statusText}`);
    }

    const data = await response.json();
    return {
      success: true,
      data,
    };
  }

  async manageTask(params) {
    const { action, task_id, title, description, status, assignee, due_date, priority } = params;
    
    const credentials = await this.getCredentials();
    const { apiKey, baseUrl } = credentials;

    let url, method, body;

    if (action === 'create') {
      url = `${baseUrl}/api/v1/tasks`;
      method = 'POST';
      body = { title, description, assignee, due_date, priority };
    } else if (action === 'update') {
      if (!task_id) {
        throw new Error('任务ID不能为空');
      }
      url = `${baseUrl}/api/v1/tasks/${task_id}`;
      method = 'PATCH';
      body = {};
      if (status) body.status = status;
      if (title) body.title = title;
      if (description) body.description = description;
    } else if (action === 'query') {
      if (!task_id) {
        throw new Error('任务ID不能为空');
      }
      url = `${baseUrl}/api/v1/tasks/${task_id}`;
      method = 'GET';
    }

    const options = {
      method,
      headers: {
        'Authorization': `Bearer ${apiKey}`,
        'Content-Type': 'application/json',
      },
    };

    if (body) {
      options.body = JSON.stringify(body);
    }

    const response = await fetch(url, options);

    if (!response.ok) {
      throw new Error(`API请求失败: ${response.statusText}`);
    }

    const data = await response.json();
    return {
      success: true,
      data,
    };
  }
}

module.exports = EnterprisePlugin;
```

## 5. OpenAI兼容格式开发

### 5.1 工具定义格式对比

主流AI平台对工具的定义格式略有不同：

```typescript
// OpenAI格式
const openAITool = {
  type: "function",
  function: {
    name: "get_weather",
    description: "获取城市天气信息",
    parameters: {
      type: "object",
      properties: {
        city: {
          type: "string",
          description: "城市名称"
        },
        unit: {
          type: "string",
          enum: ["celsius", "fahrenheit"]
        }
      },
      required: ["city"]
    }
  }
};

// Anthropic Claude格式
const claudeTool = {
  name: "get_weather",
  description: "获取城市天气信息",
  input_schema: {
    type: "object",
    properties: {
      city: {
        type: "string",
        description: "城市名称"
      },
      unit: {
        type: "string",
        enum: ["celsius", "fahrenheit"]
      }
    },
    required: ["city"]
  }
};

// Google Gemini格式
const geminiTool = {
  name: "get_weather",
  description: "获取城市天气信息",
  parameters: {
    type: "object",
    properties: {
      city: {
        type: "STRING",
        description: "城市名称"
      },
      unit: {
        type: "STRING",
        enum: ["celsius", "fahrenheit"]
      }
    },
    required: ["city"]
  }
};
```

### 5.2 统一工具注册表

```typescript
// unified-tool-registry.ts
import type { OpenAITool } from 'openai';

// 工具定义接口
interface ToolDefinition {
  name: string;
  description: string;
  parameters: {
    [key: string]: {
      type: string;
      description?: string;
      enum?: string[];
      default?: any;
      required?: boolean;
      minimum?: number;
      maximum?: number;
    };
  };
  required: string[];
}

// 工具接口
interface Tool {
  definition: ToolDefinition;
  handler: (params: any) => Promise<any>;
  validator?: (params: any) => boolean;
}

// 统一工具注册表
class UnifiedToolRegistry {
  private tools: Map<string, Tool> = new Map();

  // 注册工具
  register(tool: Tool): void {
    // 验证工具定义
    if (!this.validateDefinition(tool.definition)) {
      throw new Error(`Invalid tool definition for ${tool.definition.name}`);
    }
    this.tools.set(tool.definition.name, tool);
  }

  // 验证定义
  private validateDefinition(def: ToolDefinition): boolean {
    if (!def.name || !def.description) {
      return false;
    }
    if (!def.parameters || typeof def.parameters !== 'object') {
      return false;
    }
    return true;
  }

  // 转换为OpenAI格式
  toOpenAIFormat(): OpenAITool[] {
    return Array.from(this.tools.values()).map(tool => ({
      type: 'function',
      function: {
        name: tool.definition.name,
        description: tool.definition.description,
        parameters: {
          type: 'object',
          properties: tool.definition.parameters,
          required: tool.definition.required,
        },
      },
    }));
  }

  // 转换为Claude格式
  toClaudeFormat(): any[] {
    return Array.from(this.tools.values()).map(tool => ({
      name: tool.definition.name,
      description: tool.definition.description,
      input_schema: {
        type: 'object',
        properties: tool.definition.parameters,
        required: tool.definition.required,
      },
    }));
  }

  // 转换为Gemini格式
  toGeminiFormat(): any[] {
    return Array.from(this.tools.values()).map(tool => {
      // Gemini需要大写类型
      const params: any = { type: 'OBJECT' };
      params.properties = {};
      
      for (const [key, prop] of Object.entries(tool.definition.parameters)) {
        params.properties[key] = {
          type: prop.type.toUpperCase(),
          description: prop.description,
          enum: prop.enum,
        };
      }
      
      return {
        name: tool.definition.name,
        description: tool.definition.description,
        parameters: params,
      };
    });
  }

  // 执行工具
  async execute(toolName: string, params: any): Promise<any> {
    const tool = this.tools.get(toolName);
    
    if (!tool) {
      throw new Error(`Tool not found: ${toolName}`);
    }

    // 参数验证
    if (tool.validator && !tool.validator(params)) {
      throw new Error(`Invalid parameters for tool: ${toolName}`);
    }

    // 执行
    return await tool.handler(params);
  }

  // 列出所有工具
  listTools(): string[] {
    return Array.from(this.tools.keys());
  }

  // 获取工具定义
  getTool(name: string): Tool | undefined {
    return this.tools.get(name);
  }
}

// 创建全局注册表
export const registry = new UnifiedToolRegistry();
```

### 5.3 工具实现示例

```typescript
// tools/weather.ts
import { registry } from './unified-tool-registry';

registry.register({
  definition: {
    name: 'get_weather',
    description: '获取指定城市的天气信息，包括温度、湿度、风速等',
    parameters: {
      city: {
        type: 'string',
        description: '城市名称，支持中文和英文，如"北京"、"Shanghai"'
      },
      unit: {
        type: 'string',
        description: '温度单位',
        enum: ['celsius', 'fahrenheit'],
        default: 'celsius'
      },
      include_forecast: {
        type: 'boolean',
        description: '是否包含预报',
        default: false
      }
    },
    required: ['city']
  },
  validator: (params) => {
    if (!params.city || params.city.trim() === '') {
      return false;
    }
    return true;
  },
  handler: async (params) => {
    const { city, unit = 'celsius', include_forecast = false } = params;

    // 模拟API调用
    const response = await fetch(
      `https://api.weather.example.com/v1/weather?city=${encodeURIComponent(city)}&unit=${unit}`
    );

    if (!response.ok) {
      throw new Error(`Weather API error: ${response.statusText}`);
    }

    const data = await response.json();

    const result: any = {
      city: data.location,
      temperature: data.temp,
      feels_like: data.feels_like,
      humidity: data.humidity,
      wind_speed: data.wind_speed,
      condition: data.condition,
      updated_at: data.updated_at
    };

    if (include_forecast) {
      result.forecast = data.forecast;
    }

    return result;
  }
});

// 注册更多工具
registry.register({
  definition: {
    name: 'send_email',
    description: '发送电子邮件',
    parameters: {
      to: {
        type: 'string',
        description: '收件人邮箱'
      },
      subject: {
        type: 'string',
        description: '邮件主题'
      },
      body: {
        type: 'string',
        description: '邮件正文'
      },
      cc: {
        type: 'string',
        description: '抄送人邮箱，多个用逗号分隔'
      }
    },
    required: ['to', 'subject', 'body']
  },
  handler: async (params) => {
    const { to, subject, body, cc } = params;

    const response = await fetch('https://api.email.example.com/send', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ to, subject, body, cc })
    });

    if (!response.ok) {
      throw new Error(`Email API error: ${response.statusText}`);
    }

    return await response.json();
  }
});
```

## 6. 认证与安全

### 6.1 凭证安全存储

```typescript
// secure-credential-store.ts
import crypto from 'crypto';

interface EncryptedCredential {
  key: string;
  iv: string;
  encryptedData: string;
  tag: string;
}

class SecureCredentialStore {
  private encryptionKey: Buffer;
  
  constructor(encryptionKey: string) {
    // 密钥必须是32字节（AES-256）
    this.encryptionKey = crypto.scryptSync(encryptionKey, 'salt', 32);
  }

  // 加密凭证
  encrypt(plaintext: string): EncryptedCredential {
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipheriv('aes-256-gcm', this.encryptionKey, iv);
    
    let encrypted = cipher.update(plaintext, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    return {
      key: encrypted,
      iv: iv.toString('hex'),
      encryptedData: encrypted,
      tag: cipher.getAuthTag().toString('hex')
    };
  }

  // 解密凭证
  decrypt(encrypted: EncryptedCredential): string {
    const decipher = crypto.createDecipheriv(
      'aes-256-gcm',
      this.encryptionKey,
      Buffer.from(encrypted.iv, 'hex')
    );
    
    decipher.setAuthTag(Buffer.from(encrypted.tag, 'hex'));
    
    let decrypted = decipher.update(encrypted.encryptedData, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return decrypted;
  }

  // 保存到存储
  async save(key: string, value: string, storage: any): Promise<void> {
    const encrypted = this.encrypt(value);
    await storage.set(key, JSON.stringify(encrypted));
  }

  // 从存储读取
  async get(key: string, storage: any): Promise<string | null> {
    const data = await storage.get(key);
    if (!data) return null;
    
    const encrypted: EncryptedCredential = JSON.parse(data);
    return this.decrypt(encrypted);
  }
}
```

### 6.2 OAuth 2.0集成

```typescript
// oauth-handler.ts
interface OAuthConfig {
  clientId: string;
  clientSecret: string;
  redirectUri: string;
  authorizationUrl: string;
  tokenUrl: string;
  scopes: string[];
}

interface OAuthToken {
  access_token: string;
  refresh_token?: string;
  expires_in: number;
  token_type: string;
  scope?: string;
}

class OAuthHandler {
  private config: OAuthConfig;
  private stateStorage: Map<string, string>;

  constructor(config: OAuthConfig) {
    this.config = config;
    this.stateStorage = new Map();
  }

  // 生成授权URL
  generateAuthorizationUrl(state?: string): { url: string; state: string } {
    const finalState = state || this.generateState();
    this.stateStorage.set(finalState, Date.now().toString());

    const params = new URLSearchParams({
      client_id: this.config.clientId,
      redirect_uri: this.config.redirectUri,
      response_type: 'code',
      scope: this.config.scopes.join(' '),
      state: finalState
    });

    return {
      url: `${this.config.authorizationUrl}?${params.toString()}`,
      state: finalState
    };
  }

  // 验证state
  validateState(state: string): boolean {
    const timestamp = this.stateStorage.get(state);
    if (!timestamp) return false;
    
    // state有效期10分钟
    const age = Date.now() - parseInt(timestamp);
    if (age > 10 * 60 * 1000) {
      this.stateStorage.delete(state);
      return false;
    }
    
    this.stateStorage.delete(state);
    return true;
  }

  // 交换授权码
  async exchangeCodeForToken(code: string): Promise<OAuthToken> {
    const response = await fetch(this.config.tokenUrl, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
        'Authorization': `Basic ${Buffer.from(
          `${this.config.clientId}:${this.config.clientSecret}`
        ).toString('base64')}`
      },
      body: new URLSearchParams({
        grant_type: 'authorization_code',
        code,
        redirect_uri: this.config.redirectUri
      })
    });

    if (!response.ok) {
      const error = await response.text();
      throw new Error(`Token exchange failed: ${error}`);
    }

    return await response.json();
  }

  // 刷新Token
  async refreshToken(refreshToken: string): Promise<OAuthToken> {
    const response = await fetch(this.config.tokenUrl, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
        'Authorization': `Basic ${Buffer.from(
          `${this.config.clientId}:${this.config.clientSecret}`
        ).toString('base64')}`
      },
      body: new URLSearchParams({
        grant_type: 'refresh_token',
        refresh_token: refreshToken
      })
    });

    if (!response.ok) {
      throw new Error(`Token refresh failed: ${response.statusText}`);
    }

    return await response.json();
  }

  private generateState(): string {
    return crypto.randomBytes(16).toString('hex');
  }
}
```

## 7. 实战案例：企业AI助手插件

### 7.1 项目结构

```
enterprise-ai-assistant/
├── dify-plugin/              # Dify插件
│   ├── __init__.py
│   ├── manifest.yaml
│   ├── credentials.py
│   └── tools/
│       ├── employee.py
│       ├── task.py
│       ├── approval.py
│       └── __init__.py
│
├── n8n-nodes/               # n8n节点
│   ├── src/
│   │   ├── nodes/Enterprise/
│   │   └── credentials/
│   └── package.json
│
├── coze-plugin/             # Coze插件
│   ├── openapi.yaml
│   └── icon.png
│
├── shared/                  # 共享代码
│   ├── api-client.ts
│   ├── types.ts
│   └── auth.ts
│
└── README.md
```

### 7.2 统一API客户端

```typescript
// shared/api-client.ts
import { OAuthHandler } from './auth';

interface ApiClientConfig {
  baseUrl: string;
  auth: OAuthConfig | { apiKey: string };
}

class ApiClient {
  private baseUrl: string;
  private accessToken?: string;
  private oauthHandler?: OAuthHandler;
  private refreshToken?: string;
  private tokenExpiresAt?: number;

  constructor(config: ApiClientConfig) {
    this.baseUrl = config.baseUrl.rstrip('/');
    
    if ('authorizationUrl' in config.auth) {
      this.oauthHandler = new OAuthHandler(config.auth);
    }
  }

  // OAuth认证
  async authorize(): Promise<string> {
    if (!this.oauthHandler) {
      throw new Error('OAuth not configured');
    }
    
    const { url, state } = this.oauthHandler.generateAuthorizationUrl();
    // 返回授权URL，前端重定向用户
    return url;
  }

  // 处理OAuth回调
  async handleCallback(code: string, state: string): Promise<void> {
    if (!this.oauthHandler) {
      throw new Error('OAuth not configured');
    }
    
    if (!this.oauthHandler.validateState(state)) {
      throw new Error('Invalid state parameter');
    }
    
    const token = await this.oauthHandler.exchangeCodeForToken(code);
    this.setToken(token);
  }

  // 设置Token
  setToken(token: { access_token: string; refresh_token?: string; expires_in: number }) {
    this.accessToken = token.access_token;
    this.refreshToken = token.refresh_token;
    this.tokenExpiresAt = Date.now() + token.expires_in * 1000;
  }

  // API请求
  async request<T>(
    method: 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE',
    path: string,
    data?: any
  ): Promise<T> {
    // 检查Token是否过期
    if (this.tokenExpiresAt && Date.now() > this.tokenExpiresAt - 60000) {
      await this.refreshAccessToken();
    }

    const url = `${this.baseUrl}${path}`;
    const options: RequestInit = {
      method,
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.accessToken}`
      }
    };

    if (data && method !== 'GET') {
      options.body = JSON.stringify(data);
    }

    const response = await fetch(url, options);

    if (response.status === 401) {
      // Token过期，尝试刷新
      await this.refreshAccessToken();
      options.headers = {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.accessToken}`
      };
      const retryResponse = await fetch(url, options);
      
      if (!retryResponse.ok) {
        throw new Error(`API request failed: ${retryResponse.statusText}`);
      }
      
      return await retryResponse.json();
    }

    if (!response.ok) {
      throw new Error(`API request failed: ${response.statusText}`);
    }

    return await response.json();
  }

  private async refreshAccessToken(): Promise<void> {
    if (!this.oauthHandler || !this.refreshToken) {
      throw new Error('Cannot refresh token');
    }

    const token = await this.oauthHandler.refreshToken(this.refreshToken);
    this.setToken(token);
  }
}
```

## 8. 总结

### 插件开发速查表

| 平台 | 开发方式 | 认证方式 | 适用场景 |
|------|----------|----------|----------|
| Dify | Python类继承 | 凭证配置 | 工具扩展 |
| n8n | TypeScript节点 | 凭证系统 | 流程节点 |
| Coze | OpenAPI导入/SDK | Bearer Token | Bot扩展 |
| 自定义 | 多语言SDK | 灵活配置 | 跨平台 |

### 开发建议

1. **从简单开始**：先用OpenAPI导入测试功能
2. **复用认证**：凭证逻辑尽量复用
3. **统一工具格式**：用统一注册表支持多平台
4. **完善错误处理**：API调用要有容错
5. **添加日志**：方便调试和监控

---

## 相关资源

- [[Function_Calling与工具调用]] - 函数调用规范
- [[n8n与LLM集成]] - n8n AI能力
- [[扣子Bot开发]] - Coze Bot开发
- [[AI应用API化部署]] - API部署方案
- [[多Agent系统设计]] - 多智能体协作

---

*本文档由归愚知识系统生成 last updated: 2026-04-24*
