# order +confirm

> **前置条件：** 先阅读 [`../../opc-shared/SKILL.md`](../../opc-shared/SKILL.md) 了解认证、全局参数和数据表结构。

确认订单，将订单状态从"待处理"更新为"已确认"。

需要的scopes: ["base:app:write"]

## 命令

```bash
# 确认订单
lark-cli base app_table_records update \
  --table-id opc_orders \
  --record-id <record_id> \
  --as bot \
  --data '{
    "fields": {
      "status": "已确认",
      "updated_at": 1741564800
    }
  }'

# 快捷确认（使用脚本）
# 1. 查询订单
ORDER_RESULT=$(lark-cli base app_table_records search \
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
  }')

# 2. 提取 record_id
RECORD_ID=$(echo $ORDER_RESULT | jq -r '.data.items[0].record_id')

# 3. 确认订单
lark-cli base app_table_records update \
  --table-id opc_orders \
  --record-id $RECORD_ID \
  --as bot \
  --data '{
    "fields": {
      "status": "已确认",
      "updated_at": '$(date +%s)'
    }
  }'
```

## 参数

| 参数 | 必填 | 说明 |
|------|------|------|
| `--table-id <id>` | ✅ | 订单表的 table_id |
| `--record-id <id>` | ✅ | 订单记录的 record_id |
| `--as <identity>` | 否 | 身份类型：bot（默认）或 user |
| `--data <json>` | ✅ | 更新字段数据（JSON 格式） |
| `--dry-run` | 否 | 预览 API 调用，不执行 |

## 前置条件

确认订单前，应确保：

1. ✅ **客户信息完整** - 客户档案已创建
2. ✅ **订单明细完整** - 产品、数量、单价已填写
3. ✅ **库存充足** - 库存检查通过
4. ✅ **金额正确** - 订单总额计算无误

## 确认订单的影响

确认订单后会触发以下操作：

- ✅ **锁定库存** - 扣减可用库存，增加锁定库存
- ✅ **生成应收记录** - 在财务模块生成应收账款
- ✅ **通知相关部门** - 通知仓库、财务等部门
- ✅ **状态变更** - 订单状态从"待处理"变为"已确认"

## 输出格式

成功时返回更新后的订单记录：

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
        "status": "已确认",
        "updated_at": 1741564800
      }
    }
  }
}
```

## 提示

- **获取 record_id**：先通过 `order +query` 查询订单记录
- **库存检查**：确认前务必检查库存是否充足
- **金额核对**：确认前务必核对订单金额是否正确
- **权限控制**：只有销售主管或以上权限可以确认订单
- **不可逆操作**：订单确认后无法恢复为"待处理"状态

## 错误处理

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| `record not found` | 订单不存在 | 检查 record_id 是否正确 |
| `permission denied` | 权限不足 | 确认是否有确认订单权限 |
| `invalid status transition` | 状态转换不允许 | 只有"待处理"状态可以确认 |
| `insufficient inventory` | 库存不足 | 先检查库存，再确认订单 |
| `missing order details` | 订单明细不完整 | 补充订单明细后再确认 |

## 完整流程

### 场景 1：正常确认流程

```bash
# 1. 查询待处理订单
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

# 2. 检查库存
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

# 3. 确认订单
lark-cli base app_table_records update \
  --table-id opc_orders \
  --record-id <record_id> \
  --as bot \
  --data '{
    "fields": {
      "status": "已确认",
      "updated_at": '$(date +%s)'
    }
  }'

# 4. 更新库存（锁定库存）
# opc-inventory +update
```

### 场景 2：库存不足时处理

```bash
# 1. 检查库存
INVENTORY_RESULT=$(lark-cli base app_table_records search \
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
  }')

AVAILABLE_STOCK=$(echo $INVENTORY_RESULT | jq -r '.data.items[0].fields.available_stock')
REQUIRED_QUANTITY=100

# 2. 判断库存是否充足
if [ $AVAILABLE_STOCK -lt $REQUIRED_QUANTITY ]; then
  echo "库存不足，可用库存：$AVAILABLE_STOCK，需要：$REQUIRED_QUANTITY"
  # 通知销售和客户
  # 创建补货单
else
  # 库存充足，确认订单
  lark-cli base app_table_records update \
    --table-id opc_orders \
    --record-id <record_id> \
    --as bot \
    --data '{
      "fields": {
        "status": "已确认",
        "updated_at": '$(date +%s)'
      }
    }'
fi
```

## 后续操作

确认订单后，可以继续：

1. 创建发货单：`opc-order +ship`
2. 生成发票：`opc-order +invoice`
3. 确认收款：`opc-order +payment-receive`

## 参考

- [order SKILL](../SKILL.md) -- 订单管理全部命令
- [order +query](opc-order-query.md) -- 查询订单
- [order +create](opc-order-create.md) -- 创建订单
- [order +ship](opc-order-ship.md) -- 创建发货单
- [inventory SKILL](../../opc-inventory/SKILL.md) -- 库存管理
- [opc-shared](../../opc-shared/SKILL.md) -- 认证和全局参数
