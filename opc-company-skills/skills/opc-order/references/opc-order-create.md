# order +create

> **前置条件：** 先阅读 [`../../opc-shared/SKILL.md`](../../opc-shared/SKILL.md) 了解认证、全局参数和数据表结构。

创建新订单，自动生成订单编号。

需要的scopes: ["base:app:write"]

## 命令

```bash
# 创建订单（完整示例）
lark-cli base app_table_records create \
  --table-id opc_orders \
  --as bot \
  --data '{
    "fields": {
      "order_id": "ORD-20260417-001",
      "customer_id": "CUS-20260417-001",
      "order_date": 1741564800,
      "total_amount": 100000,
      "status": "待处理",
      "sales_owner": "张三",
      "created_at": 1741564800,
      "updated_at": 1741564800
    }
  }'

# 快捷创建（使用脚本）
ORDER_DATE=$(date +%Y%m%d)
ORDER_ID="ORD-${ORDER_DATE}-001"
lark-cli base app_table_records create \
  --table-id opc_orders \
  --as bot \
  --data "{
    \"fields\": {
      \"order_id\": \"${ORDER_ID}\",
      \"customer_id\": \"${CUSTOMER_ID}\",
      \"order_date\": $(date +%s),
      \"total_amount\": ${TOTAL_AMOUNT},
      \"status\": \"待处理\",
      \"sales_owner\": \"${SALES_OWNER}\",
      \"created_at\": $(date +%s),
      \"updated_at\": $(date +%s)
    }
  }"
```

## 参数

| 参数 | 必填 | 说明 |
|------|------|------|
| `--table-id <id>` | ✅ | 订单表的 table_id |
| `--as <identity>` | 否 | 身份类型：bot（默认）或 user |
| `--data <json>` | ✅ | 订单数据（JSON 格式） |
| `--dry-run` | 否 | 预览 API 调用，不执行 |

## 数据字段

### 必填字段

| 字段 | 类型 | 说明 | 示例 |
|------|------|------|------|
| `order_id` | text | 订单编号 | ORD-20260417-001 |
| `customer_id` | text | 客户编号 | CUS-20260417-001 |
| `order_date` | datetime | 订单日期 | Unix 时间戳 |
| `total_amount` | number | 订单总金额（分） | 100000（1000.00元） |
| `status` | select | 订单状态 | 待处理 |
| `sales_owner` | text | 销售负责人 | 张三 |
| `created_at` | datetime | 创建时间 | Unix 时间戳 |
| `updated_at` | datetime | 更新时间 | Unix 时间戳 |

## 订单编号生成规则

格式：`ORD-YYYYMMDD-XXX`

- **日期部分**：`YYYYMMDD`（如 20260417）
- **序号部分**：三位数，从 001 开始
- **重置规则**：每天从 001 重新开始

示例：
- `ORD-20260417-001` - 2026年4月17日的第1个订单
- `ORD-20260417-002` - 2026年4月17日的第2个订单

## 订单状态说明

| 状态 | 说明 |
|------|------|
| **待处理** | 订单已创建，待确认 |
| **已确认** | 订单已确认，待发货 |
| **已发货** | 已发货，待收款 |
| **已完成** | 订单完成 |
| **已取消** | 订单已取消 |

## 金额计算规则

- **单位**：分（避免浮点误差）
- **计算公式**：总金额 = Σ(产品单价 × 数量)
- **示例**：100.00 元 = 10000 分

## 输出格式

成功时返回创建的订单记录：

```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "record": {
      "record_id": "rectxxxxxx",
      "fields": {
        "order_id": "ORD-20260417-001",
        "customer_id": "CUS-20260417-001",
        "total_amount": 100000,
        "status": "待处理"
      }
    }
  }
}
```

## 提示

- **获取 table_id**：使用 `lark-cli base tables list --as bot` 查询
- **时间戳格式**：使用 `date +%s` 获取当前 Unix 时间戳
- **客户验证**：创建前建议先验证客户是否存在
- **库存检查**：创建后应立即检查库存是否充足

## 后续操作

创建订单后，可以继续：

1. 检查库存：`opc-inventory +check`
2. 添加订单明细：创建 `opc_order_items` 记录
3. 确认订单：`opc-order +confirm`
4. 创建发货单：`opc-order +ship`

## 错误处理

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| `customer not found` | 客户不存在 | 先创建客户档案 |
| `table not found` | 表不存在 | 先创建数据表 |
| `permission denied` | 权限不足 | 检查 `base:app:write` scope |
| `invalid amount` | 金额格式错误 | 确保金额为整数（分） |

## 参考

- [order SKILL](../SKILL.md) -- 订单管理全部命令
- [order +query](opc-order-query.md) -- 查询订单
- [order +confirm](opc-order-confirm.md) -- 确认订单
- [customer SKILL](../../opc-customer/SKILL.md) -- 客户管理
- [inventory SKILL](../../opc-inventory/SKILL.md) -- 库存管理
- [opc-shared](../../opc-shared/SKILL.md) -- 认证和全局参数
