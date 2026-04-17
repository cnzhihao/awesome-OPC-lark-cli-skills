# inventory +adjust

> **前置条件：** 先阅读 [`../../opc-shared/SKILL.md`](../../opc-shared/SKILL.md) 了解认证、全局参数和数据表结构。

调整库存，用于盘点差异、损耗等情况。

需要的scopes: ["base:app:write"]

## 命令

```bash
# 库存调整
lark-cli base app_table_records create \
  --table-id opc_inventory_adjustments \
  --as bot \
  --data '{
    "fields": {
      "adjustment_id": "ADJ-20260417-001",
      "product_id": "PROD-001",
      "adjustment_type": "损耗",
      "quantity": -10,
      "reason": "产品损坏",
      "created_at": 1741564800
    }
  }'
```

## 调整类型

| 类型 | 说明 |
|------|------|
| 盘点 | 实际库存与系统库存差异 |
| 损耗 | 产品损坏、过期 |
| 盘盈 | 实际库存多于系统库存 |

## 参考

- [inventory SKILL](../SKILL.md)
