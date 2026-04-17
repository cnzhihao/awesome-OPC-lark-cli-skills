---
name: opc-inventory
version: 1.0.0
description: "OPC 库存管理系统：库存查询、补货提醒、库存预警。支持实时库存查询、库存扣减、补货建议、库存预警通知。"
metadata:
  category: "erp"
  requires:
    bins: ["lark-cli"]
  cliHelp: "lark-cli base --help"
---

# OPC 库存管理系统

**CRITICAL — 开始前 MUST 先用 Read 工具读取 [`../opc-shared/SKILL.md`](../opc-shared/SKILL.md)，其中包含认证、权限处理**

## 核心概念

### 库存数据模型

```
库存 (Inventory)
├── 产品信息：产品编号、产品名称、规格型号
├── 库存数量：当前库存、可用库存、锁定库存
├── 库存预警：最低库存、最高库存、补货点
└── 补货信息：补货周期、补货批量、供应商
```

### 库存流转

```
采购入库 → 库存增加
销售出库 → 库存减少
退货 → 库存恢复
报废 → 库存扣减
```

### 数据表结构

**表名：`opc_inventory`**

| 字段名 | 类型 | 说明 | 必填 |
|--------|------|------|------|
| product_id | text | 产品编号 | ✅ |
| product_name | text | 产品名称 | ✅ |
| specification | text | 规格型号 | |
| current_stock | number | 当前库存 | ✅ |
| available_stock | number | 可用库存 | ✅ |
| locked_stock | number | 锁定库存 | ✅ |
| min_stock | number | 最低库存 | ✅ |
| max_stock | number | 最高库存 | ✅ |
| reorder_point | number | 补货点 | ✅ |
| warehouse | text | 仓库位置 | ✅ |
| last_updated | datetime | 最后更新时间 | ✅ |

## 典型工作流

### 库存检查流程

```
1. 查询库存
   → opc-inventory +check

2. 判断是否需要补货
   → 如果 current_stock <= reorder_point

3. 创建补货单
   → opc-inventory +restock

4. 通知采购员
   → 发送飞书消息
```

### 库存预警流程

```
1. 查询库存预警
   → opc-inventory +alert

2. 分析预警原因
   → 销售异常 / 供应延迟 / 计划不当

3. 制定补货计划
   → opc-inventory +restock

4. 跟踪补货进度
   → 定期查询补货状态
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
| [`+check`](references/opc-inventory-check.md) | 查询库存 |
| [`+update`](references/opc-inventory-update.md) | 更新库存 |
| [`+alert`](references/opc-inventory-alert.md) | 查询库存预警 |
| [`+restock`](references/opc-inventory-restock.md) | 创建补货单 |
| [`+adjust`](references/opc-inventory-adjust.md) | 库存调整 |

## 库存字段说明

| 字段 | 说明 | 计算公式 |
|------|------|---------|
| **current_stock** | 当前库存总量 | physical_stock |
| **available_stock** | 可用库存 | current_stock - locked_stock |
| **locked_stock** | 锁定库存（订单占用） | order_reserved |
| **min_stock** | 最低库存 | 安全库存线 |
| **max_stock** | 最高库存 | 仓库容量限制 |
| **reorder_point** | 补货点 | 触发补货的库存阈值 |

## 业务规则

### 库存扣减规则

- **订单确认时**：扣减 available_stock，增加 locked_stock
- **发货时**：扣减 current_stock 和 locked_stock
- **取消订单时**：恢复 available_stock，减少 locked_stock

### 补货规则

- **补货触发**：current_stock <= reorder_point
- **补货批量**：建议补货到 max_stock
- **补货周期**：根据供应商交货期确定

### 库存预警规则

| 预警级别 | 条件 | 处理措施 |
|---------|------|---------|
| 🔴 严重 | current_stock < min_stock | 紧急补货 |
| 🟡 警告 | min_stock <= current_stock <= reorder_point | 计划补货 |
| 🟢 正常 | current_stock > reorder_point | 正常运营 |

## API Resources

### 查询库存

```bash
lark-cli base app_table_records search \
  --table-id <table_id> \
  --as bot \
  --data '{
    "filter": {
      "conditions": [{
        "field_name": "product_id",
        "operator": "is",
        "value": ["PROD-001"]
      }]
    }
  }'
```

### 更新库存

```bash
lark-cli base app_table_records update \
  --table-id <table_id> \
  --record-id <record_id> \
  --as bot \
  --data '{
    "fields": {
      "current_stock": 1000,
      "available_stock": 800,
      "locked_stock": 200,
      "last_updated": 1741564800
    }
  }'
```

### 查询库存预警

```bash
lark-cli base app_table_records search \
  --table-id <table_id> \
  --as bot \
  --data '{
    "filter": {
      "logic": "and",
      "conditions": [{
        "field_name": "current_stock",
        "operator": "lessThan",
        "value": [100]
      }]
    }
  }'
```

## 库存计算示例

### 场景 1：订单确认

```
初始状态：
- current_stock: 1000
- available_stock: 1000
- locked_stock: 0

订单确认（订单数量 200）：
- current_stock: 1000（不变）
- available_stock: 800（1000 - 200）
- locked_stock: 200（0 + 200）
```

### 场景 2：发货

```
发货前：
- current_stock: 1000
- available_stock: 800
- locked_stock: 200

发货（发货数量 200）：
- current_stock: 800（1000 - 200）
- available_stock: 800（不变）
- locked_stock: 0（200 - 200）
```

### 场景 3：补货

```
补货前：
- current_stock: 100
- reorder_point: 200
- max_stock: 1000

补货建议：
- 补货数量 = max_stock - current_stock = 1000 - 100 = 900
```

## 权限表

| 操作 | 所需 scope |
|------|-----------|
| 查询库存 | `base:app:read` |
| 更新库存 | `base:app:write` |
| 库存调整 | `base:app:write` |
| 补货操作 | `base:app:write` |

## 常见场景

### 场景 1：每日库存检查

```bash
# 1. 查询库存预警
opc-inventory +alert

# 2. 分析预警原因
# 3. 创建补货单
opc-inventory +restock

# 4. 通知采购员
```

### 场景 2：订单库存检查

```bash
# 1. 查询产品库存
opc-inventory +check --product-id "PROD-001"

# 2. 判断是否充足
# 3. 如果不足，通知销售和客户
# 4. 调整发货计划
```

### 场景 3：月度库存盘点

```bash
# 1. 导出库存数据
lark-cli base app_table_records export --table-id opc_inventory

# 2. 与实际库存对比
# 3. 调整库存差异
opc-inventory +adjust

# 4. 分析差异原因
# 5. 优化库存管理
```

## 相关 Skill

- 订单管理：`../opc-order/SKILL.md`
- 采购管理（待开发）：`../opc-procurement/SKILL.md`
- 财务管理：`../opc-finance/SKILL.md`
