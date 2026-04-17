# inventory +restock

> **前置条件：** 先阅读 [`../../opc-shared/SKILL.md`](../../opc-shared/SKILL.md) 了解认证、全局参数和数据表结构。

创建补货单，补充库存。

需要的scopes: ["base:app:write"]

## 命令

```bash
# 创建补货单
lark-cli base app_table_records create \
  --table-id opc_restock_orders \
  --as bot \
  --data '{
    "fields": {
      "restock_id": "RST-20260417-001",
      "product_id": "PROD-001",
      "quantity": 900,
      "status": "待审批",
      "created_at": 1741564800
    }
  }'
```

## 补货建议

```
补货数量 = max_stock - current_stock
```

## 参考

- [inventory SKILL](../SKILL.md)
- [inventory +check](opc-inventory-check.md)
