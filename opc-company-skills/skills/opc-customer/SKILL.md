---
name: opc-customer
version: 1.0.0
description: "OPC 客户管理系统：客户档案管理、联系人管理、合同管理、客户跟进记录。支持客户创建、查询、更新、删除，以及合同全生命周期管理。"
metadata:
  category: "erp"
  requires:
    bins: ["lark-cli"]
  cliHelp: "lark-cli base --help"
---

# OPC 客户管理系统

**CRITICAL — 开始前 MUST 先用 Read 工具读取 [`../opc-shared/SKILL.md`](../opc-shared/SKILL.md)，其中包含认证、权限处理**

## 核心概念

### 客户数据模型

```
客户 (Customer)
├── 基本信息：公司名称、行业、规模、地址
├── 联系信息：电话、邮箱、网址
├── 联系人 (Contacts)：姓名、职位、手机、邮箱
├── 合同 (Contracts)：合同号、金额、起止日期、状态
└── 跟进记录 (Activities)：沟通时间、内容、下次跟进
```

### 数据表结构

**表名：`opc_customers`**

| 字段名 | 类型 | 说明 | 必填 |
|--------|------|------|------|
| customer_id | text | 客户编号（自动生成，如 CUS-001） | ✅ |
| company_name | text | 公司名称 | ✅ |
| industry | text | 所属行业 | |
| company_size | text | 公司规模 | |
| address | text | 公司地址 | |
| phone | text | 公司电话 | |
| email | text | 公司邮箱 | |
| website | text | 公司网址 | |
| status | select | 客户状态（潜在/意向/成交/流失） | ✅ |
| level | select | 客户等级（A/B/C） | ✅ |
| created_at | datetime | 创建时间 | ✅ |
| updated_at | datetime | 更新时间 | ✅ |
| sales_owner | text | 销售负责人 | ✅ |

## Shortcuts（推荐优先使用）

| Shortcut | 说明 |
|----------|------|
| [`+create`](references/opc-customer-create.md) | 创建新客户档案 |
| [`+query`](references/opc-customer-query.md) | 查询客户（支持多条件筛选） |
| [`+update`](references/opc-customer-update.md) | 更新客户信息 |
| [`+contract-create`](references/opc-customer-contract.md) | 创建客户合同 |

## API Resources

### 获取表 ID

```bash
lark-cli base tables list --as bot | grep opc_customers
```

### 创建客户记录

```bash
lark-cli base app_table_records create \
  --table-id <table_id> \
  --as bot \
  --data '{
    "fields": {
      "customer_id": "CUS-001",
      "company_name": "示例公司",
      "industry": "互联网",
      "status": "潜在",
      "level": "A",
      "sales_owner": "张三"
    }
  }'
```

### 查询客户记录

```bash
lark-cli base app_table_records search \
  --table-id <table_id> \
  --as bot \
  --data '{
    "filter": {
      "conditions": [
        {
          "field_name": "company_name",
          "operator": "contains",
          "value": ["示例"]
        }
      ]
    }
  }'
```

### 更新客户记录

```bash
lark-cli base app_table_records update \
  --table-id <table_id> \
  --record-id <record_id> \
  --as bot \
  --data '{
    "fields": {
      "status": "成交",
      "level": "A"
    }
  }'
```

## 客户状态流转

```
潜在 → 意向 → 成交
  ↓        ↓
  └─────── 流失
```

## 业务规则

### 客户编号生成

- 格式：`CUS-YYYYMMDD-XXX`
- 示例：`CUS-20260417-001`
- 每天从 001 开始编号

### 客户等级评定

| 等级 | 标准 |
|------|------|
| A | 年采购额 > 100 万 |
| B | 年采购额 50-100 万 |
| C | 年采购额 < 50 万 |

### 数据权限

- 销售只能查看和编辑自己的客户
- 销售主管可以查看团队所有客户
- 管理员可以查看所有客户

## 权限表

| 操作 | 所需 scope |
|------|-----------|
| 创建客户 | `base:app:write` |
| 查询客户 | `base:app:read` |
| 更新客户 | `base:app:write` |
| 删除客户 | `base:app:write` |

## 常见场景

### 场景 1：新客户注册

```bash
# 1. 创建客户档案
opc-customer +create

# 2. 添加联系人
# 3. 创建首单合同
opc-customer +contract-create
```

### 场景 2：客户跟进

```bash
# 1. 查询客户信息
opc-customer +query --company-name "示例公司"

# 2. 记录跟进活动
# 3. 设置下次跟进提醒
```

### 场景 3：客户分析

```bash
# 按行业统计客户分布
# 按等级统计客户价值
# 分析客户流失率
```

## 相关 Skill

- 订单管理：`../opc-order/SKILL.md`
- 财务管理：`../opc-finance/SKILL.md`
