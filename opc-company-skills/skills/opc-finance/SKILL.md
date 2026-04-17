---
name: opc-finance
version: 1.0.0
description: "OPC 财务管理系统：应收应付管理、财务报表、成本分析。支持客户应收账款查询、供应商应付账款管理、财务报表生成、成本核算。"
metadata:
  category: "erp"
  requires:
    bins: ["lark-cli"]
  cliHelp: "lark-cli base --help"
---

# OPC 财务管理系统

**CRITICAL — 开始前 MUST 先用 Read 工具读取 [`../opc-shared/SKILL.md`](../opc-shared/SKILL.md)，其中包含认证、权限处理**

## 核心概念

### 财务数据模型

```
财务 (Finance)
├── 应收账款 (Receivables)：客户欠款、账期分析、回款跟踪
├── 应付账款 (Payables)：供应商欠款、付款计划、发票管理
├── 成本核算 (Costing)：产品成本、运营成本、利润分析
└── 财务报表 (Reports)：资产负债表、损益表、现金流量表
```

### 财务流转

```
销售产生应收账款
采购产生应付账款
收款减少应收账款
付款减少应付账款
```

### 数据表结构

**表名：`opc_receivables`**（应收账款）

| 字段名 | 类型 | 说明 | 必填 |
|--------|------|------|------|
| receivable_id | text | 应收编号 | ✅ |
| customer_id | text | 客户编号 | ✅ |
| order_id | text | 订单编号 | ✅ |
| amount | number | 应收金额（分） | ✅ |
| paid_amount | number | 已收金额（分） | ✅ |
| balance | number | 余额（分） | ✅ |
| due_date | datetime | 应收日期 | ✅ |
| status | select | 状态 | ✅ |
| created_at | datetime | 创建时间 | ✅ |

**表名：`opc_payables`**（应付账款）

| 字段名 | 类型 | 说明 | 必填 |
|--------|------|------|------|
| payable_id | text | 应付编号 | ✅ |
| supplier_id | text | 供应商编号 | ✅ |
| purchase_order_id | text | 采购订单号 | ✅ |
| amount | number | 应付金额（分） | ✅ |
| paid_amount | number | 已付金额（分） | ✅ |
| balance | number | 余额（分） | ✅ |
| due_date | datetime | 应付日期 | ✅ |
| status | select | 状态 | ✅ |
| created_at | datetime | 创建时间 | ✅ |

## 典型工作流

### 应收账款管理

```
1. 订单完成后生成应收记录
   → 订单系统触发

2. 查询应收账款
   → opc-finance +receivable

3. 收款确认
   → opc-finance +payment-receive

4. 更新应收记录
   → 自动更新余额和状态
```

### 应付账款管理

```
1. 采购订单生成应付记录
   → 采购系统触发

2. 查询应付账款
   → opc-finance +payable

3. 付款审批
   → 飞书审批流程

4. 付款确认
   → opc-finance +payment

5. 更新应付记录
   → 自动更新余额和状态
```

### 财务报表生成

```
1. 导出财务数据
   → lark-cli base app_table_records export

2. 数据汇总和分析
   → 生成财务报表

3. 可视化展示
   → 飞书仪表盘
```

### CRITICAL — 首次使用任何命令前先查 `-h`

无论是 Shortcut 还是原生 API，**首次调用前必须先运行 `-h` 查看可用参数**：

```bash
# Shortcut
lark-cli base app_table_records search -h

# 原生 API
lark-cli base app_table_records -h
```

## Shortcuts（推荐优先使用）

| Shortcut | 说明 |
|----------|------|
| [`+receivable`](references/opc-finance-receivable.md) | 查询应收账款 |
| [`+payable`](references/opc-finance-payable.md) | 查询应付账款 |
| [`+payment-receive`](references/opc-finance-payment-receive.md) | 收款确认 |
| [`+payment`](references/opc-finance-payment.md) | 付款确认 |
| [`+report`](references/opc-finance-report.md) | 生成财务报表 |
| [`+aging`](references/opc-finance-aging.md) | 账龄分析 |

## 财务状态说明

### 应收账款状态

| 状态 | 说明 |
|------|------|
| **待收款** | 未到应收日期 |
| **应收中** | 已到应收日期，未收款 |
| **部分收款** | 部分收款 |
| **已结清** | 全部收款 |
| **逾期** | 超过应收日期未收款 |

### 应付账款状态

| 状态 | 说明 |
|------|------|
| **待付款** | 未到应付日期 |
| **应付中** | 已到应付日期，未付款 |
| **部分付款** | 部分付款 |
| **已结清** | 全部付款 |
| **逾期** | 超过应付日期未付款 |

## 账龄分析

### 应收账款账龄

| 账龄区间 | 说明 | 风险等级 |
|---------|------|---------|
| **0-30天** | 正常账期 | 🟢 低风险 |
| **31-60天** | 逾期1个月内 | 🟡 中风险 |
| **61-90天** | 逾期2个月内 | 🟠 高风险 |
| **90天以上** | 逾期3个月以上 | 🔴 严重风险 |

### 应付账款账龄

| 账龄区间 | 说明 | 优先级 |
|---------|------|--------|
| **0-30天** | 正常账期 | 低 |
| **31-60天** | 逾期1个月内 | 中 |
| **60天以上** | 逾期2个月以上 | 高 |

## API Resources

### 查询应收账款

```bash
lark-cli base app_table_records search \
  --table-id opc_receivables \
  --as bot \
  --data '{
    "filter": {
      "conditions": [{
        "field_name": "customer_id",
        "operator": "is",
        "value": ["CUS-20260417-001"]
      }]
    }
  }'
```

### 查询应付账款

```bash
lark-cli base app_table_records search \
  --table-id opc_payables \
  --as bot \
  --data '{
    "filter": {
      "conditions": [{
        "field_name": "supplier_id",
        "operator": "is",
        "value": ["SUP-001"]
      }]
    }
  }'
```

### 更新应收记录

```bash
lark-cli base app_table_records update \
  --table-id opc_receivables \
  --record-id <record_id> \
  --as bot \
  --data '{
    "fields": {
      "paid_amount": 50000,
      "balance": 50000,
      "status": "部分收款"
    }
  }'
```

## 业务规则

### 金额计算规则

- **单位**：分（避免浮点误差）
- **余额计算**：balance = amount - paid_amount
- **状态更新**：根据余额自动更新状态

### 收款确认流程

1. 收到客户款项
2. 确认收款金额
3. 更新应收记录
4. 通知销售和客户

### 付款确认流程

1. 发起付款审批
2. 审批通过后付款
3. 确认付款金额
4. 更新应付记录
5. 通知供应商

## 权限表

| 操作 | 所需 scope | 审批要求 |
|------|-----------|---------|
| 查询应收 | `base:app:read` | - |
| 查询应付 | `base:app:read` | - |
| 收款确认 | `base:app:write` | - |
| 付款确认 | `base:app:write` | 财务审批 |
| 生成报表 | `base:app:read` | - |

## 常见场景

### 场景 1：每日应收对账

```bash
# 1. 查询今日到期应收
opc-finance +receivable --due-date today

# 2. 通知销售跟进
# 3. 记录收款情况
opc-finance +payment-receive

# 4. 更新应收记录
```

### 场景 2：每周应付计划

```bash
# 1. 查询本周到期应付
opc-finance +payable --due-week this-week

# 2. 制定付款计划
# 3. 发起付款审批
# 4. 执行付款
opc-finance +payment
```

### 场景 3：月度财务报表

```bash
# 1. 导出财务数据
lark-cli base app_table_records export --table-id opc_receivables
lark-cli base app_table_records export --table-id opc_payables

# 2. 生成财务报表
opc-finance +report --period 2026-04

# 3. 账龄分析
opc-finance +aging

# 4. 利润分析
opc-finance +profit
```

## 相关 Skill

- 订单管理：`../opc-order/SKILL.md`
- 客户管理：`../opc-customer/SKILL.md`
- 采购管理（待开发）：`../opc-procurement/SKILL.md`
