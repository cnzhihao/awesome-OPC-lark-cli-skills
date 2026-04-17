# customer +query

> **前置条件：** 先阅读 [`../../opc-shared/SKILL.md`](../../opc-shared/SKILL.md) 了解认证、全局参数和数据表结构。

查询客户信息，支持多条件筛选。

需要的scopes: ["base:app:read"]

## 命令

```bash
# 查询所有客户
lark-cli base app_table_records search \
  --table-id opc_customers \
  --as bot \
  --data '{
    "filter": {
      "conjunction": "and",
      "conditions": []
    }
  }'

# 按客户名称查询
lark-cli base app_table_records search \
  --table-id opc_customers \
  --as bot \
  --data '{
    "filter": {
      "conditions": [{
        "field_name": "company_name",
        "operator": "contains",
        "value": ["科技"]
      }]
    }
  }'

# 按客户编号查询
lark-cli base app_table_records search \
  --table-id opc_customers \
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

# 按状态查询（如：查询所有潜在客户）
lark-cli base app_table_records search \
  --table-id opc_customers \
  --as bot \
  --data '{
    "filter": {
      "conditions": [{
        "field_name": "status",
        "operator": "is",
        "value": ["潜在"]
      }]
    }
  }'

# 多条件组合查询（如：查询A类潜在客户）
lark-cli base app_table_records search \
  --table-id opc_customers \
  --as bot \
  --data '{
    "filter": {
      "logic": "and",
      "conditions": [
        {
          "field_name": "status",
          "operator": "is",
          "value": ["潜在"]
        },
        {
          "field_name": "level",
          "operator": "is",
          "value": ["A"]
        }
      ]
    }
  }'
```

## 参数

| 参数 | 必填 | 说明 |
|------|------|------|
| `--table-id <id>` | ✅ | 客户表的 table_id |
| `--as <identity>` | 否 | 身份类型：bot（默认）或 user |
| `--data <json>` | ✅ | 查询条件（JSON 格式） |
| `--page-size <num>` | 否 | 每页记录数（默认100） |
| `--page-token <token>` | 否 | 分页令牌 |

## 筛选条件

### 支持的操作符

| 操作符 | 说明 | 适用字段 |
|--------|------|---------|
| `is` | 等于 | 文本、选择、数字 |
| `contains` | 包含 | 文本 |
| `notContains` | 不包含 | 文本 |
| `greaterThan` | 大于 | 数字、日期 |
| `lessThan` | 小于 | 数字、日期 |
| `greaterThanOrEqual` | 大于等于 | 数字、日期 |
| `lessThanOrEqual` | 小于等于 | 数字、日期 |
| `isEmpty` | 为空 | 所有类型 |
| `isNotEmpty` | 不为空 | 所有类型 |

### 可筛选字段

| 字段 | 类型 | 推荐操作符 |
|------|------|-----------|
| `customer_id` | text | is, contains |
| `company_name` | text | is, contains |
| `industry` | text | is, contains |
| `status` | select | is |
| `level` | select | is |
| `sales_owner` | text | is, contains |
| `created_at` | datetime | greaterThan, lessThan |

## 输出格式

成功时返回客户列表：

```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "items": [{
      "record_id": "rectxxxxxx",
      "fields": {
        "customer_id": "CUS-20260417-001",
        "company_name": "科技创新有限公司",
        "industry": "互联网",
        "status": "潜在",
        "level": "A",
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

### 场景 1：查询某销售的所有客户

```bash
lark-cli base app_table_records search \
  --table-id opc_customers \
  --as bot \
  --data '{
    "filter": {
      "conditions": [{
        "field_name": "sales_owner",
        "operator": "is",
        "value": ["张三"]
      }]
    }
  }'
```

### 场景 2：查询最近7天创建的客户

```bash
SEVEN_DAYS_AGO=$(date -d '-7 days' +%s)
lark-cli base app_table_records search \
  --table-id opc_customers \
  --as bot \
  --data "{
    \"filter\": {
      \"conditions\": [{
        \"field_name\": \"created_at\",
        \"operator\": \"greaterThanOrEqual\",
        \"value\": [${SEVEN_DAYS_AGO}]
      }]
    }
  }"
```

### 场景 3：查询未成交的A类客户

```bash
lark-cli base app_table_records search \
  --table-id opc_customers \
  --as bot \
  --data '{
    "filter": {
      "logic": "and",
      "conditions": [
        {
          "field_name": "level",
          "operator": "is",
          "value": ["A"]
        },
        {
          "field_name": "status",
          "operator": "isNot",
          "value": ["成交"]
        }
      ]
    }
  }'
```

## 提示

- **获取 table_id**：使用 `lark-cli base tables list --as bot` 查询
- **分页查询**：当结果超过100条时，使用 `page_token` 继续查询
- **性能优化**：尽量使用索引字段（customer_id, status, level）进行筛选
- **结果导出**：可结合 `jq` 等工具处理结果，或导出到文件

## 错误处理

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| `table not found` | 表不存在 | 检查 table_id 是否正确 |
| `permission denied` | 权限不足 | 检查 `base:app:read` scope |
| `invalid filter` | 筛选条件错误 | 检查字段名和操作符是否正确 |
| `field not found` | 字段不存在 | 检查字段名是否正确 |

## 参考

- [customer SKILL](../SKILL.md) -- 客户管理全部命令
- [customer +create](opc-customer-create.md) -- 创建客户
- [customer +update](opc-customer-update.md) -- 更新客户
- [opc-shared](../../opc-shared/SKILL.md) -- 认证和全局参数
