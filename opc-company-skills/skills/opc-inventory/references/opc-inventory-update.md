# inventory +update

> **前置条件：** 先阅读 [`../../opc-shared/SKILL.md`](../../opc-shared/SKILL.md) 了解认证、全局参数和数据表结构。

更新库存信息，包括库存扣减、补充、调整等操作。

需要的scopes: ["base:app:write"]

## 命令

```bash
# 更新库存
lark-cli base app_table_records update \
  --table-id opc_inventory \
  --record-id <record_id> \
  --as bot \
  --data '{
    "fields": {
      "current_stock": 900,
      "available_stock": 700,
      "locked_stock": 200,
      "last_updated": 1741564800
    }
  }'

# 订单确认时锁定库存
lark-cli base app_table_records update \
  --table-id opc_inventory \
  --record-id <record_id> \
  --as bot \
  --data '{
    "fields": {
      "available_stock": 600,
      "locked_stock": 400,
      "last_updated": 1741564800
    }
  }'

# 发货时扣减库存
lark-cli base app_table_records update \
  --table-id opc_inventory \
  --record-id <record_id> \
  --as bot \
  --data '{
    "fields": {
      "current_stock": 700,
      "available_stock": 700,
      "locked_stock": 0,
      "last_updated": 1741564800
    }
  }'
```

## 库存更新场景

### 场景 1：订单确认（锁定库存）

```
变化：
- current_stock: 1000 → 1000（不变）
- available_stock: 1000 → 800（减少）
- locked_stock: 0 → 200（增加）
```

### 场景 2：发货（扣减库存）

```
变化：
- current_stock: 1000 → 800（减少）
- available_stock: 800 → 800（不变）
- locked_stock: 200 → 0（减少）
```

### 场景 3：补货入库

```
变化：
- current_stock: 800 → 1800（增加）
- available_stock: 800 → 1800（增加）
- locked_stock: 0 → 0（不变）
```

## 参考

- [inventory SKILL](../SKILL.md)
- [inventory +check](opc-inventory-check.md)
- [order SKILL](../../opc-order/SKILL.md)
