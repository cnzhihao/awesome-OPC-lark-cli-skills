# inventory +alert

> **前置条件：** 先阅读 [`../../opc-shared/SKILL.md`](../../opc-shared/SKILL.md) 了解认证、全局参数和数据表结构。

查询库存预警，识别需要补货的产品。

需要的scopes: ["base:app:read"]

## 命令

```bash
# 查询所有库存预警
lark-cli base app_table_records search \
  --table-id opc_inventory \
  --as bot \
  --data '{
    "filter": {
      "conditions": [{
        "field_name": "current_stock",
        "operator": "lessThanOrEqual",
        "value": [200]
      }]
    }
  }'
```

## 预警级别

| 级别 | 条件 | 处理措施 |
|------|------|---------|
| 🔴 严重 | current_stock < min_stock | 紧急补货 |
| 🟡 警告 | min_stock ≤ current_stock ≤ reorder_point | 计划补货 |

## 参考

- [inventory SKILL](../SKILL.md)
- [inventory +restock](opc-inventory-restock.md)
