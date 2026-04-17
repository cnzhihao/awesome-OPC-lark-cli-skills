# order +query

> **前置条件：** 先阅读 [`../../opc-shared/SKILL.md`](../../opc-shared/SKILL.md) 了解认证、全局参数和数据表结构。

查询订单信息，支持多条件筛选。

需要的scopes: ["base:app:read"]

## 命令

```bash
# 查询所有订单
lark-cli base app_table_records search \
  --table-id opc_orders \
  --as bot \
  --data '{
    "filter": {
      "conditions": []
    }
  }'

# 按订单号查询
lark-cli base app_table_records search \
  --table-id opc_orders \
  --as bot \
  --data '{
    "filter": {
      "conditions": [{
        "field_name": "order_id",
        "operator": "is",
        "value": ["ORD-20260417-001"]
      }]
    }
  }'

# 按客户查询
lark-cli base app_table_records search \
  --table-id opc_orders \
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

# 按状态查询（如：查询待处理订单）
lark-cli base app_table_records search \
  --table-id opc_orders \
  --as bot \
  --data '{
    "filter": {
      "conditions": [{
        "field_name": "status",
        "operator": "is",
        "value": ["待处理"]
      }]
    }
  }'

# 按日期范围查询（如：查询本月订单）
lark-cli base app_table_records search \
  --table-id opc_orders \
  --as bot \
  --data '{
    "filter": {
      "logic": "and",
      "conditions": [
        {
          "field_name": "order_date",
          "operator": "greaterThanOrEqual",
          "value": [1741564800]
        },
        {
          "field_name": "order_date",
          "operator": "lessThan",
          "value": [1744243200]
        }
      ]
    }
  }'

# 多条件组合查询（如：查询某客户的已发货订单）
lark-cli base app_table_records search \
  --table-id opc_orders \
  --as bot \
  --data '{
    "filter": {
      "logic": "and",
      "conditions": [
        {
          "field_name": "customer_id",
          "operator": "is",
          "value": ["CUS-20260417-001"]
        },
        {
          "field_name": "status",
          "operator": "is",
          "value": ["已发货"]
        }
      ]
    }
  }'
```

## 参数

| 参数 | 必填 | 说明 |
|------|------|------|
| `--table-id <id>` | ✅ | 订单表的 table_id |
| `--as <identity>` | 否 | 身份类型：bot（默认）或 user |
| `--data <json>` | ✅ | 查询条件（JSON 格式） |
| `--page-size <num>` | 否 | 每页记录数（默认100） |
| `--page-token <token>` | 否 | 分页令牌 |

## 筛选条件

### 可筛选字段

| 字段 | 类型 | 推荐操作符 |
|------|------|-----------|
| `order_id` | text | is, contains |
| `customer_id` | text | is |
| `order_date` | datetime | greaterThan, lessThan, greaterThanOrEqual, lessThanOrEqual |
| `total_amount` | number | greaterThan, lessThan |
| `status` | select | is, isNot |
| `sales_owner` | text | is, contains |
| `created_at` | datetime | greaterThan, lessThan |

## 输出格式

成功时返回订单列表：

```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "items": [{
      "record_id": "rectxxxxxx",
      "fields": {
        "order_id": "ORD-20260417-001",
        "customer_id": "CUS-20260417-001",
        "order_date": 1741564800,
        "total_amount": 100000,
        "status": "待处理",
        "sales_owner": "张三",
        "created_at": 1741564800
      }
    }],
    "has_more": false,
    "page_token": ""
  }
}
```

## 常见查询场景

### 场景 1：查询待处理订单

```bash
lark-cli base app_table_records search \
  --table-id opc_orders \
  --as bot \
  --data '{
    "filter": {
      "conditions": [{
        "field_name": "status",
        "operator": "is",
        "value": ["待处理"]
      }]
    }
  }'
```

### 场景 2：查询今日订单

```bash
TODAY_START=$(date -d 'today 00:00:00' +%s)
TODAY_END=$(date -d 'today 23:59:59' +%s)
lark-cli base app_table_records search \
  --table-id opc_orders \
  --as bot \
  --data "{
    \"filter\": {
      \"logic\": \"and\",
      \"conditions\": [
        {
          \"field_name\": \"order_date\",
          \"operator\": \"greaterThanOrEqual\",
          \"value\": [${TODAY_START}]
        },
        {
          \"field_name\": \"order_date\",
          \"operator\": \"lessThanOrEqual\",
          \"value\": [${TODAY_END}]
        }
      ]
    }
  }"
```

### 场景 3：查询大额订单（>10万）

```bash
lark-cli base app_table_records search \
  --table-id opc_orders \
  --as bot \
  --data '{
    "filter": {
      "conditions": [{
        "field_name": "total_amount",
        "operator": "greaterThan",
        "value": [1000000]
      }]
    }
  }'
```

### 场景 4：查询某销售的待收款订单

```bash
lark-cli base app_table_records search \
  --table-id opc_orders \
  --as bot \
  --data '{
    "filter": {
      "logic": "and",
      "conditions": [
        {
          "field_name": "sales_owner",
          "operator": "is",
          "value": ["张三"]
        },
        {
          "field_name": "status",
          "operator": "is",
          "value": ["已发货"]
        }
      ]
    }
  }'
```

## 提示

- **获取 table_id**：使用 `lark-cli base tables list --as bot` 查询
- **分页查询**：当结果超过100条时，使用 `page_token` 继续查询
- **性能优化**：尽量使用索引字段（order_id, customer_id, status）进行筛选
- **结果排序**：可结合 `jq` 等工具对结果进行排序处理

## 错误处理

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| `table not found` | 表不存在 | 检查 table_id 是否正确 |
| `permission denied` | 权限不足 | 检查 `base:app:read` scope |
| `invalid filter` | 筛选条件错误 | 检查字段名和操作符是否正确 |
| `field not found` | 字段不存在 | 检查字段名是否正确 |

## 参考

- [order SKILL](../SKILL.md) -- 订单管理全部命令
- [order +create](opc-order-create.md) -- 创建订单
- [order +confirm](opc-order-confirm.md) -- 确认订单
- [customer SKILL](../../opc-customer/SKILL.md) -- 客户管理
- [opc-shared](../../opc-shared/SKILL.md) -- 认证和全局参数
