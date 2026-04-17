# customer +update

> **前置条件：** 先阅读 [`../../opc-shared/SKILL.md`](../../opc-shared/SKILL.md) 了解认证、全局参数和数据表结构。

更新客户信息。

需要的scopes: ["base:app:write"]

## 命令

```bash
# 更新客户状态
lark-cli base app_table_records update \
  --table-id opc_customers \
  --record-id <record_id> \
  --as bot \
  --data '{
    "fields": {
      "status": "成交",
      "updated_at": 1741564800
    }
  }'

# 更新客户等级
lark-cli base app_table_records update \
  --table-id opc_customers \
  --record-id <record_id> \
  --as bot \
  --data '{
    "fields": {
      "level": "A",
      "updated_at": 1741564800
    }
  }'

# 更新客户基本信息
lark-cli base app_table_records update \
  --table-id opc_customers \
  --record-id <record_id> \
  --as bot \
  --data '{
    "fields": {
      "industry": "互联网",
      "company_size": "100-500人",
      "address": "北京市朝阳区",
      "phone": "010-12345678",
      "email": "contact@example.com",
      "website": "https://example.com",
      "updated_at": 1741564800
    }
  }'

# 更新销售负责人
lark-cli base app_table_records update \
  --table-id opc_customers \
  --record-id <record_id> \
  --as bot \
  --data '{
    "fields": {
      "sales_owner": "李四",
      "updated_at": 1741564800
    }
  }'
```

## 参数

| 参数 | 必填 | 说明 |
|------|------|------|
| `--table-id <id>` | ✅ | 客户表的 table_id |
| `--record-id <id>` | ✅ | 客户记录的 record_id |
| `--as <identity>` | 否 | 身份类型：bot（默认）或 user |
| `--data <json>` | ✅ | 更新字段数据（JSON 格式） |
| `--dry-run` | 否 | 预览 API 调用，不执行 |

## 可更新字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `company_name` | text | 公司名称 |
| `industry` | text | 所属行业 |
| `company_size` | text | 公司规模 |
| `address` | text | 公司地址 |
| `phone` | text | 公司电话 |
| `email` | text | 公司邮箱 |
| `website` | text | 公司网址 |
| `status` | select | 客户状态 |
| `level` | select | 客户等级 |
| `sales_owner` | text | 销售负责人 |
| `updated_at` | datetime | 更新时间 |

## 状态流转规则

```
潜在 → 意向 → 成交
  ↓        ↓
  └─────── 流失
```

**可执行的状态转换：**

| 当前状态 | 可转换为 | 说明 |
|---------|---------|------|
| 潜在 | 意向、流失 | 有明确需求或明确拒绝 |
| 意向 | 成交、流失 | 签署合同或放弃跟进 |
| 成交 | 流失 | 客户停止合作 |
| 流失 | 潜在、意向 | 重新激活客户 |

## 等级调整规则

| 当前等级 | 可调整为 | 条件 |
|---------|---------|------|
| C | B | 年采购额达到 50-100 万 |
| B | A | 年采购额超过 100 万 |
| A | B | 年采购额降至 50-100 万 |
| B | C | 年采购额低于 50 万 |

## 输出格式

成功时返回更新后的客户记录：

```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "record": {
      "record_id": "rectxxxxxx",
      "fields": {
        "customer_id": "CUS-20260417-001",
        "company_name": "科技创新有限公司",
        "status": "成交",
        "level": "A",
        "updated_at": 1741564800
      }
    }
  }
}
```

## 提示

- **获取 record_id**：先通过 `customer +query` 查询客户记录
- **部分更新**：只需传递需要更新的字段，不需要传递所有字段
- **时间戳**：更新时务必更新 `updated_at` 字段
- **权限控制**：销售只能更新自己的客户，主管可更新团队客户
- **状态变更**：重要状态变更（如流失）建议添加备注说明原因

## 错误处理

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| `record not found` | 记录不存在 | 检查 record_id 是否正确 |
| `permission denied` | 权限不足 | 检查是否有权限更新该客户 |
| `invalid field` | 字段不存在 | 检查字段名是否正确 |
| `status transition not allowed` | 状态转换不允许 | 检查状态流转规则 |

## 常见场景

### 场景 1：客户成交

```bash
# 1. 查询客户
CUSTOMER_RECORD=$(lark-cli base app_table_records search \
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
  }')

# 2. 提取 record_id
RECORD_ID=$(echo $CUSTOMER_RECORD | jq -r '.data.items[0].record_id')

# 3. 更新状态
lark-cli base app_table_records update \
  --table-id opc_customers \
  --record-id $RECORD_ID \
  --as bot \
  --data '{
    "fields": {
      "status": "成交",
      "updated_at": '$(date +%s)'
    }
  }'
```

### 场景 2：调整客户等级

```bash
# 根据采购额调整客户等级
# 假设年采购额超过100万升级为A级
lark-cli base app_table_records update \
  --table-id opc_customers \
  --record-id <record_id> \
  --as bot \
  --data '{
    "fields": {
      "level": "A",
      "updated_at": '$(date +%s)'
    }
  }'
```

### 场景 3：变更销售负责人

```bash
# 当销售离职或重新分配客户时
lark-cli base app_table_records update \
  --table-id opc_customers \
  --record-id <record_id> \
  --as bot \
  --data '{
    "fields": {
      "sales_owner": "李四",
      "updated_at": '$(date +%s)'
    }
  }'
```

## 参考

- [customer SKILL](../SKILL.md) -- 客户管理全部命令
- [customer +query](opc-customer-query.md) -- 查询客户
- [customer +create](opc-customer-create.md) -- 创建客户
- [opc-shared](../../opc-shared/SKILL.md) -- 认证和全局参数
