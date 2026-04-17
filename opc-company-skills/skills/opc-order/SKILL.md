---
name: opc-order
version: 1.0.0
description: "OPC 订单管理系统：订单创建、发货处理、开票、回款管理。支持订单全生命周期管理，包括订单创建、库存检查、发货通知、发票生成、收款确认。"
metadata:
  category: "erp"
  requires:
    bins: ["lark-cli"]
  cliHelp: "lark-cli base --help"
---

# OPC 订单管理系统

**CRITICAL — 开始前 MUST 先用 Read 工具读取 [`../opc-shared/SKILL.md`](../opc-shared/SKILL.md)，其中包含认证、权限处理**

## 核心概念

### 订单数据模型

```
订单 (Order)
├── 基本信息：订单号、客户、订单日期、总金额
├── 订单明细 (Order Items)：产品、数量、单价、小计
├── 发货信息 (Shipment)：发货时间、物流单号、收货地址
├── 发票信息 (Invoice)：发票号、开票日期、发票类型
└── 收款信息 (Payment)：收款日期、收款金额、收款方式
```

### 订单状态流转

```
待处理 → 已确认 → 已发货 → 已完成
         ↓                      ↓
       已取消              已退款
```

### 数据表结构

**表名：`opc_orders`**

| 字段名 | 类型 | 说明 | 必填 |
|--------|------|------|------|
| order_id | text | 订单编号（自动生成） | ✅ |
| customer_id | text | 客户编号 | ✅ |
| order_date | datetime | 订单日期 | ✅ |
| total_amount | number | 订单总金额（分） | ✅ |
| status | select | 订单状态 | ✅ |
| sales_owner | text | 销售负责人 | ✅ |
| created_at | datetime | 创建时间 | ✅ |
| updated_at | datetime | 更新时间 | ✅ |

**表名：`opc_order_items`**（订单明细）

| 字段名 | 类型 | 说明 | 必填 |
|--------|------|------|------|
| item_id | text | 明细编号 | ✅ |
| order_id | text | 订单编号 | ✅ |
| product_id | text | 产品编号 | ✅ |
| quantity | number | 数量 | ✅ |
| unit_price | number | 单价（分） | ✅ |
| subtotal | number | 小计（分） | ✅ |

## 典型工作流

### 新订单处理流程

```
1. 创建订单
   → opc-order +create

2. 检查库存
   → opc-inventory +check

3. 确认订单
   → opc-order +confirm

4. 创建发货单
   → opc-order +ship

5. 生成发票
   → opc-order +invoice

6. 收款确认
   → opc-order +payment-receive
```

### CRITICAL — 首次使用任何命令前先查 `-h`

无论是 Shortcut 还是原生 API，**首次调用前必须先运行 `-h` 查看可用参数**：

```bash
# Shortcut
lark-cli base app_table_records create -h

# 原生 API
lark-cli base app_table_records -h
```

## Shortcuts（推荐优先使用）

| Shortcut | 说明 |
|----------|------|
| [`+create`](references/opc-order-create.md) | 创建新订单 |
| [`+query`](references/opc-order-query.md) | 查询订单 |
| [`+confirm`](references/opc-order-confirm.md) | 确认订单 |
| [`+ship`](references/opc-order-ship.md) | 创建发货单 |
| [`+invoice`](references/opc-order-invoice.md) | 生成发票 |
| [`+payment-receive`](references/opc-order-payment.md) | 收款确认 |

## 订单状态说明

| 状态 | 说明 | 可执行操作 |
|------|------|-----------|
| **待处理** | 订单已创建，待确认 | 确认、取消 |
| **已确认** | 订单已确认，待发货 | 发货 |
| **已发货** | 已发货，待收款 | 确认收款 |
| **已完成** | 订单完成 | - |
| **已取消** | 订单已取消 | - |
| **已退款** | 已退款 | - |

## API Resources

### 获取表 ID

```bash
lark-cli base tables list --as bot | grep opc_orders
```

### 创建订单

```bash
lark-cli base app_table_records create \
  --table-id <table_id> \
  --as bot \
  --data '{
    "fields": {
      "order_id": "ORD-20260417-001",
      "customer_id": "CUS-20260417-001",
      "order_date": 1741564800,
      "total_amount": 100000,
      "status": "待处理",
      "sales_owner": "张三"
    }
  }'
```

### 查询订单

```bash
lark-cli base app_table_records search \
  --table-id <table_id> \
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

## 订单编号生成规则

格式：`ORD-YYYYMMDD-XXX`

- **日期部分**：`YYYYMMDD`（如 20260417）
- **序号部分**：三位数，从 001 开始
- **重置规则**：每天从 001 重新开始

## 业务规则

### 订单金额计算

- 总金额 = Σ(产品单价 × 数量)
- 金额单位：分（避免浮点误差）
- 示例：100.00 元 = 10000 分

### 库存扣减

- 订单确认时扣减库存
- 取消订单时恢复库存
- 发货时不影响库存

### 发货规则

- 仅"已确认"状态可发货
- 发货后状态变为"已发货"
- 需填写物流单号

### 开票规则

- 仅"已发货"状态可开票
- 支持增值税普通发票和专用发票
- 开票后自动通知客户

## 权限表

| 操作 | 所需 scope |
|------|-----------|
| 创建订单 | `base:app:write` |
| 查询订单 | `base:app:read` |
| 确认订单 | `base:app:write` |
| 发货 | `base:app:write` |
| 开票 | `base:app:write` |
| 收款确认 | `base:app:write` |

## 常见场景

### 场景 1：新订单创建

```
1. 创建订单
   → opc-order +create

2. 检查库存
   → opc-inventory +check

3. 确认订单
   → opc-order +confirm
```

### 场景 2：订单发货

```
1. 查询订单状态
   → opc-order +query

2. 创建发货单
   → opc-order +ship

3. 通知客户
   → 发送飞书消息
```

### 场景 3：订单收款

```
1. 查询待收款订单
   → opc-order +query --status "已发货"

2. 确认收款
   → opc-order +payment-receive

3. 更新财务记录
   → opc-finance +payment-record
```

## 相关 Skill

- 客户管理：`../opc-customer/SKILL.md`
- 库存管理：`../opc-inventory/SKILL.md`
- 财务管理：`../opc-finance/SKILL.md`
