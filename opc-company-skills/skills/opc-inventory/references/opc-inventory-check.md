# inventory +check

> **前置条件：** 先阅读 [`../../opc-shared/SKILL.md`](../../opc-shared/SKILL.md) 了解认证、全局参数和数据表结构。

查询库存信息，包括当前库存、可用库存、锁定库存等。

需要的scopes: ["base:app:read"]

## 命令

```bash
# 查询所有库存
lark-cli base app_table_records search \
  --table-id opc_inventory \
  --as bot \
  --data '{
    "filter": {
      "conditions": []
    }
  }'

# 按产品编号查询
lark-cli base app_table_records search \
  --table-id opc_inventory \
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

# 按产品名称查询
lark-cli base app_table_records search \
  --table-id opc_inventory \
  --as bot \
  --data '{
    "filter": {
      "conditions": [{
        "field_name": "product_name",
        "operator": "contains",
        "value": ["产品"]
      }]
    }
  }'

# 查询库存不足的产品
lark-cli base app_table_records search \
  --table-id opc_inventory \
  --as bot \
  --data '{
    "filter": {
      "conditions": [{
        "field_name": "current_stock",
        "operator": "lessThan",
        "value": [100]
      }]
    }
  }'

# 查询库存预警产品
lark-cli base app_table_records search \
  --table-id opc_inventory \
  --as bot \
  --data '{
    "filter": {
      "logic": "and",
      "conditions": [
        {
          "field_name": "current_stock",
          "operator": "lessThanOrEqual",
          "value": [200]
        },
        {
          "field_name": "reorder_point",
          "operator": "greaterThanOrEqual",
          "value": [200]
        }
      ]
    }
  }'
```

## 参数

| 参数 | 必填 | 说明 |
|------|------|------|
| `--table-id <id>` | ✅ | 库存表的 table_id |
| `--as <identity>` | 否 | 身份类型：bot（默认）或 user |
| `--data <json>` | ✅ | 查询条件（JSON 格式） |
| `--page-size <num>` | 否 | 每页记录数（默认100） |
| `--page-token <token>` | 否 | 分页令牌 |

## 库存字段说明

| 字段 | 类型 | 说明 |
|------|------|------|
| `product_id` | text | 产品编号 |
| `product_name` | text | 产品名称 |
| `current_stock` | number | 当前库存总量 |
| `available_stock` | number | 可用库存（可销售的库存） |
| `locked_stock` | number | 锁定库存（订单占用） |
| `min_stock` | number | 最低库存（安全库存） |
| `max_stock` | number | 最高库存（仓库容量） |
| `reorder_point` | number | 补货点（触发补货的库存阈值） |
| `warehouse` | text | 仓库位置 |
| `last_updated` | datetime | 最后更新时间 |

## 库存关系

```
current_stock = available_stock + locked_stock

available_stock = current_stock - locked_stock
```

## 输出格式

成功时返回库存列表：

```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "items": [{
      "record_id": "rectxxxxxx",
      "fields": {
        "product_id": "PROD-001",
        "product_name": "示例产品",
        "current_stock": 1000,
        "available_stock": 800,
        "locked_stock": 200,
        "min_stock": 100,
        "max_stock": 1000,
        "reorder_point": 200,
        "warehouse": "北京仓",
        "last_updated": 1741564800
      }
    }],
    "has_more": false,
    "page_token": ""
  }
}
```

## 常见查询场景

### 场景 1：检查订单库存是否充足

```bash
# 查询产品库存
PRODUCT_ID="PROD-001"
REQUIRED_QUANTITY=100

INVENTORY_RESULT=$(lark-cli base app_table_records search \
  --table-id opc_inventory \
  --as bot \
  --data '{
    "filter": {
      "conditions": [{
        "field_name": "product_id",
        "operator": "is",
        "value": ["'${PRODUCT_ID}'"]
      }]
    }
  }')

AVAILABLE_STOCK=$(echo $INVENTORY_RESULT | jq -r '.data.items[0].fields.available_stock')

if [ $AVAILABLE_STOCK -ge $REQUIRED_QUANTITY ]; then
  echo "库存充足，可用库存：$AVAILABLE_STOCK"
else
  echo "库存不足，可用库存：$AVAILABLE_STOCK，需要：$REQUIRED_QUANTITY"
fi
```

### 场景 2：查询库存预警

```bash
# 查询需要补货的产品
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
  }' | jq -r '.data.items[] | "\(.fields.product_id): \(.fields.current_stock) / \(.fields.reorder_point)"'
```

### 场景 3：库存汇总统计

```bash
# 统计总库存价值
lark-cli base app_table_records search \
  --table-id opc_inventory \
  --as bot \
  --data '{
    "filter": {
      "conditions": []
    }
  }' | jq -r '[.data.items[].fields.current_stock] | add'
```

## 库存预警级别

| 级别 | 条件 | 处理措施 | 颜色标识 |
|------|------|---------|---------|
| 🔴 严重 | current_stock < min_stock | 紧急补货 | 红色 |
| 🟡 警告 | min_stock ≤ current_stock ≤ reorder_point | 计划补货 | 黄色 |
| 🟢 正常 | current_stock > reorder_point | 正常运营 | 绿色 |

## 库存健康度评估

### 健康库存

```
current_stock > reorder_point
available_stock > 订单需求
```

### 需要关注

```
min_stock ≤ current_stock ≤ reorder_point
```

### 库存异常

```
current_stock < min_stock
```

## 提示

- **获取 table_id**：使用 `lark-cli base tables list --as bot` 查询
- **实时库存**：库存数据实时更新，但可能有延迟
- **多仓库**：如果有多个仓库，需要指定 warehouse 字段
- **库存锁定**：订单确认时锁定库存，发货时扣减库存

## 错误处理

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| `product not found` | 产品不存在 | 检查 product_id 是否正确 |
| `table not found` | 表不存在 | 先创建数据表 |
| `permission denied` | 权限不足 | 检查 `base:app:read` scope |
| `invalid filter` | 筛选条件错误 | 检查字段名和操作符 |

## 库存状态说明

### 正常库存

```
current_stock = 1000
available_stock = 800
locked_stock = 200
reorder_point = 200

状态：正常（current_stock > reorder_point）
```

### 库存预警

```
current_stock = 180
available_stock = 180
locked_stock = 0
reorder_point = 200

状态：警告（current_stock ≤ reorder_point）
```

### 库存严重不足

```
current_stock = 50
available_stock = 50
locked_stock = 0
min_stock = 100
reorder_point = 200

状态：严重（current_stock < min_stock）
```

## 参考

- [inventory SKILL](../SKILL.md) -- 库存管理全部命令
- [inventory +update](opc-inventory-update.md) -- 更新库存
- [inventory +alert](opc-inventory-alert.md) -- 查询库存预警
- [inventory +restock](opc-inventory-restock.md) -- 创建补货单
- [order SKILL](../../opc-order/SKILL.md) -- 订单管理
- [opc-shared](../../opc-shared/SKILL.md) -- 认证和全局参数
