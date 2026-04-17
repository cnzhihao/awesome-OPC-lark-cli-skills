# order +ship

> **前置条件：** 先阅读 [`../../opc-shared/SKILL.md`](../../opc-shared/SKILL.md) 了解认证、全局参数和数据表结构。

创建发货单，将订单状态从"已确认"更新为"已发货"。

需要的scopes: ["base:app:write"]

## 命令

```bash
# 创建发货单
lark-cli base app_table_records create \
  --table-id opc_shipments \
  --as bot \
  --data '{
    "fields": {
      "shipment_id": "SHP-20260417-001",
      "order_id": "ORD-20260417-001",
      "customer_id": "CUS-20260417-001",
      "ship_date": 1741564800,
      "tracking_number": "SF1234567890",
      "carrier": "顺丰速运",
      "shipping_address": "北京市朝阳区某某街道",
      "status": "已发货",
      "created_at": 1741564800
    }
  }'

# 同时更新订单状态
lark-cli base app_table_records update \
  --table-id opc_orders \
  --record-id <order_record_id> \
  --as bot \
  --data '{
    "fields": {
      "status": "已发货",
      "updated_at": 1741564800
    }
  }'

# 快捷发货（使用脚本）
SHIPMENT_DATE=$(date +%Y%m%d)
SHIPMENT_ID="SHP-${SHIPMENT_DATE}-001"
lark-cli base app_table_records create \
  --table-id opc_shipments \
  --as bot \
  --data "{
    \"fields\": {
      \"shipment_id\": \"${SHIPMENT_ID}\",
      \"order_id\": \"${ORDER_ID}\",
      \"customer_id\": \"${CUSTOMER_ID}\",
      \"ship_date\": $(date +%s),
      \"tracking_number\": \"${TRACKING_NUMBER}\",
      \"carrier\": \"${CARRIER}\",
      \"shipping_address\": \"${ADDRESS}\",
      \"status\": \"已发货\",
      \"created_at\": $(date +%s)
    }
  }"
```

## 参数

| 参数 | 必填 | 说明 |
|------|------|------|
| `--table-id <id>` | ✅ | 发货表的 table_id |
| `--as <identity>` | 否 | 身份类型：bot（默认）或 user |
| `--data <json>` | ✅ | 发货单数据（JSON 格式） |
| `--dry-run` | 否 | 预览 API 调用，不执行 |

## 数据字段

### 必填字段

| 字段 | 类型 | 说明 | 示例 |
|------|------|------|------|
| `shipment_id` | text | 发货单号 | SHP-20260417-001 |
| `order_id` | text | 订单编号 | ORD-20260417-001 |
| `customer_id` | text | 客户编号 | CUS-20260417-001 |
| `ship_date` | datetime | 发货日期 | Unix 时间戳 |
| `tracking_number` | text | 物流单号 | SF1234567890 |
| `carrier` | text | 物流公司 | 顺丰速运 |
| `shipping_address` | text | 收货地址 | 北京市朝阳区某某街道 |
| `status` | select | 发货状态 | 已发货/已签收/已退回 |
| `created_at` | datetime | 创建时间 | Unix 时间戳 |

### 可选字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `recipient_name` | text | 收货人姓名 |
| `recipient_phone` | text | 收货人电话 |
| `shipping_method` | select | 配送方式 |
| `notes` | text | 备注说明 |

## 发货单号生成规则

格式：`SHP-YYYYMMDD-XXX`

- **日期部分**：`YYYYMMDD`（如 20260417）
- **序号部分**：三位数，从 001 开始
- **重置规则**：每天从 001 重新开始

示例：
- `SHP-20260417-001` - 2026年4月17日的第1个发货单
- `SHP-20260417-002` - 2026年4月17日的第2个发货单

## 物流公司代码

| 代码 | 物流公司 |
|------|---------|
| SF | 顺丰速运 |
| YTO | 圆通速递 |
| STO | 申通快递 |
| ZTO | 中通快递 |
| EMS | EMS |
| JD | 京东物流 |

## 发货状态说明

| 状态 | 说明 |
|------|------|
| **已发货** | 货物已交付物流公司 |
| **运输中** | 货物在运输途中 |
| **已签收** | 客户已签收 |
| **已退回** | 货物被退回 |
| **异常** | 运输异常 |

## 输出格式

成功时返回创建的发货单记录：

```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "record": {
      "record_id": "rectxxxxxx",
      "fields": {
        "shipment_id": "SHP-20260417-001",
        "order_id": "ORD-20260417-001",
        "tracking_number": "SF1234567890",
        "status": "已发货"
      }
    }
  }
}
```

## 提示

- **获取 table_id**：使用 `lark-cli base tables list --as bot` 查询
- **订单验证**：发货前先确认订单状态为"已确认"
- **库存扣减**：发货时需要扣减实际库存
- **物流跟踪**：记录物流单号便于后续跟踪
- **客户通知**：发货后应及时通知客户

## 业务规则

### 发货条件

- ✅ 订单状态必须为"已确认"
- ✅ 库存充足（可用库存 >= 订单数量）
- ✅ 收货地址完整
- ✅ 物流信息完整

### 库存处理

发货时的库存处理：

```
发货前：
- current_stock: 1000
- available_stock: 800
- locked_stock: 200

发货后（发货数量 200）：
- current_stock: 800（扣减）
- available_stock: 800（不变）
- locked_stock: 0（释放）
```

### 发货通知

发货后应通知：

1. **客户** - 发送发货通知和物流单号
2. **销售** - 通知订单已发货
3. **财务** - 通知可以开具发票

## 错误处理

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| `order not found` | 订单不存在 | 检查 order_id 是否正确 |
| `invalid order status` | 订单状态不允许发货 | 只有"已确认"状态可以发货 |
| `insufficient inventory` | 库存不足 | 检查库存是否充足 |
| `table not found` | 表不存在 | 先创建数据表 |
| `permission denied` | 权限不足 | 检查 `base:app:write` scope |

## 完整流程

### 场景 1：正常发货流程

```bash
# 1. 查询已确认订单
ORDER_RESULT=$(lark-cli base app_table_records search \
  --table-id opc_orders \
  --as bot \
  --data '{
    "filter": {
      "conditions": [{
        "field_name": "status",
        "operator": "is",
        "value": ["已确认"]
      }]
    }
  }')

# 2. 提取订单信息
ORDER_ID=$(echo $ORDER_RESULT | jq -r '.data.items[0].fields.order_id')
CUSTOMER_ID=$(echo $ORDER_RESULT | jq -r '.data.items[0].fields.customer_id')
ORDER_RECORD_ID=$(echo $ORDER_RESULT | jq -r '.data.items[0].record_id')

# 3. 创建发货单
SHIPMENT_DATE=$(date +%Y%m%d)
SHIPMENT_ID="SHP-${SHIPMENT_DATE}-001"
lark-cli base app_table_records create \
  --table-id opc_shipments \
  --as bot \
  --data "{
    \"fields\": {
      \"shipment_id\": \"${SHIPMENT_ID}\",
      \"order_id\": \"${ORDER_ID}\",
      \"customer_id\": \"${CUSTOMER_ID}\",
      \"ship_date\": $(date +%s),
      \"tracking_number\": \"SF1234567890\",
      \"carrier\": \"顺丰速运\",
      \"status\": \"已发货\",
      \"created_at\": $(date +%s)
    }
  }"

# 4. 更新订单状态
lark-cli base app_table_records update \
  --table-id opc_orders \
  --record-id $ORDER_RECORD_ID \
  --as bot \
  --data '{
    "fields": {
      "status": "已发货",
      "updated_at": '$(date +%s)'
    }
  }'

# 5. 通知客户
# 发送飞书消息或邮件
```

### 场景 2：批量发货

```bash
# 查询所有已确认订单
lark-cli base app_table_records search \
  --table-id opc_orders \
  --as bot \
  --data '{
    "filter": {
      "conditions": [{
        "field_name": "status",
        "operator": "is",
        "value": ["已确认"]
      }]
    }
  }' | jq -r '.data.items[] | @json' | while read -r order; do
  # 解析订单信息
  ORDER_ID=$(echo $order | jq -r '.fields.order_id')
  CUSTOMER_ID=$(echo $order | jq -r '.fields.customer_id')
  RECORD_ID=$(echo $order | jq -r '.record_id')

  # 创建发货单
  # ... 发货逻辑
done
```

## 后续操作

发货后，可以继续：

1. 生成发票：`opc-order +invoice`
2. 跟踪物流：查询物流信息
3. 确认收款：`opc-order +payment-receive`

## 参考

- [order SKILL](../SKILL.md) -- 订单管理全部命令
- [order +query](opc-order-query.md) -- 查询订单
- [order +confirm](opc-order-confirm.md) -- 确认订单
- [order +invoice](opc-order-invoice.md) -- 生成发票
- [inventory SKILL](../../opc-inventory/SKILL.md) -- 库存管理
- [opc-shared](../../opc-shared/SKILL.md) -- 认证和全局参数
