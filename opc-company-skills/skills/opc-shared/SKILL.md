---
name: opc-shared
version: 1.0.0
description: "OPC 公司经营管理系统 - 共享基础：应用配置初始化、认证登录、身份切换、权限管理、安全规则。所有 OPC Skills 的基础依赖。"
metadata:
  category: "erp"
  requires:
    bins: ["lark-cli"]
---

# OPC 系统共享规则

本 Skill 是 OPC 公司经营管理系统的**共享基础模块**，提供认证、配置、权限等核心功能。所有其他 OPC Skills 都依赖本模块。

## 核心概念

### OPC 公司经营系统

基于飞书套件构建的**全方位企业经营管理系统**，涵盖：

- **客户管理**（Customer）：客户档案、联系人、合同管理
- **订单管理**（Order）：订单创建、发货、开票、回款
- **库存管理**（Inventory）：库存查询、补货提醒、库存预警
- **财务管理**（Finance）：应收应付、财务报表、成本分析
- **人事管理**（HR）：考勤、薪资、招聘

### 数据存储架构

```
飞书多维表格 (Base)
├── opc_customers      # 客户表
├── opc_orders         # 订单表
├── opc_inventory      # 库存表
├── opc_finance        # 财务表
└── opc_employees      # 员工表
```

所有业务数据存储在**飞书多维表格**中，通过 lark-cli 操作。

## 配置初始化

### 首次使用

```bash
# 1. 安装飞书 CLI
npm install -g @larksuite/cli

# 2. 配置应用凭证
lark-cli config init

# 3. 登录授权
lark-cli auth login --recommend
```

### 数据表初始化

首次使用 OPC 系统前，需创建数据表：

```bash
# 创建多维表格（在飞书中手动创建或使用脚本）
# 表结构定义见各个子 Skill 的 references 文档
```

## 认证与权限

### 身份类型

| 身份 | 用途 |
|------|------|
| `--as user` | 员工操作自己的订单、客户 |
| `--as bot` | 系统自动化操作 |

### 权限管理

- 所有 Skills 需要 **Base 多维表格的读写权限**
- 财务相关操作需要额外的 **财务审批流程**
- 敏感操作（删除、退款）需要 **二次确认**

### 权限不足处理

遇到 `permission_violations` 错误时：

1. 检查是否已登录：`lark-cli auth status`
2. 确认已授予 `base:app:read` 和 `base:app:write` 权限
3. 检查飞书开发者后台是否开通对应 scope

## 安全规则

### 输入验证

- 所有用户输入必须经过验证，防止 SQL 注入
- 金额字段必须格式化为标准货币格式
- 日期字段必须使用 ISO 8601 格式

### 操作审计

- 所有修改操作需记录操作人、操作时间
- 敏感操作需记录原因和审批人

### 数据隔离

- 不同客户的数据严格隔离
- 财务数据仅授权人员可见

## 错误处理

### 常见错误

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| `permission_violations` | 权限不足 | 检查 scope 配置 |
| `table not found` | 数据表不存在 | 先创建数据表 |
| `record not found` | 记录不存在 | 检查 record_id |
| `invalid field` | 字段不存在 | 检查字段名 |

## 通用命令

### 查询数据表

```bash
lark-cli base tables list --as bot
```

### 查询表结构

```bash
lark-cli schema base.tables.get
```

### 创建记录

```bash
lark-cli base app_table_records create \
  --table-id <table_id> \
  --data '{"fields": {"字段名": "值"}}'
```

## 下一步

根据业务需求选择对应的 Skill：

- 客户管理：`../opc-customer/SKILL.md`
- 订单管理：`../opc-order/SKILL.md`
- 库存管理：`../opc-inventory/SKILL.md`
- 财务管理：`../opc-finance/SKILL.md`
- 人事管理：`../opc-hr/SKILL.md`
