# order +payment-receive

> **前置条件：** 先阅读 [`../../opc-shared/SKILL.md`](../../opc-shared/SKILL.md) 了解认证、全局参数和数据表结构。

确认收款，记录订单收款信息。

需要的scopes: ["base:app:write"]

## 命令

```bash
# 创建收款记录
lark-cli base app_table_records create \
  --table-id opc_payments \
  --as bot \
  --data '{
    "fields": {
      "payment_id": "PAY-20260417-001",
      "order_id": "ORD-20260417-001",
      "customer_id": "CUS-20260417-001",
      "payment_amount": 113000,
      "payment_method": "银行转账",
      "payment_date": 1741564800,
      "status": "已收款",
      "notes": "货款已收",
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
      "status": "已完成",
      "updated_at": 1741564800
    }
  }'

# 快捷收款（使用脚本）
PAYMENT_DATE=$(date +%Y%m%d)
PAYMENT_ID="PAY-${PAYMENT_DATE}-001"
lark-cli base app_table_records create \
  --table-id opc_payments \
  --as bot \
  --data "{
    \"fields\": {
      \"payment_id\": \"${PAYMENT_ID}\",
      \"order_id\": \"${ORDER_ID}\",
      \"customer_id\": \"${CUSTOMER_ID}\",
      \"payment_amount\": ${AMOUNT},
      \"payment_method\": \"银行转账\",
      \"payment_date\": $(date +%s),
      \"status\": \"已收款\",
      \"created_at\": $(date +%s)
    }
  }"
```

## 参数

| 参数 | 必填 | 说明 |
|------|------|------|
| `--table-id <id>` | ✅ | 收款表的 table_id |
| `--as <identity>` | 否 | 身份类型：bot（默认）或 user |
| `--data <json>` | ✅ | 收款数据（JSON 格式） |
| `--dry-run` | 否 | 预览 API 调用，不执行 |

## 数据字段

### 必填字段

| 字段 | 类型 | 说明 | 示例 |
|------|------|------|------|
| `payment_id` | text | 收款编号 | PAY-20260417-001 |
| `order_id` | text | 订单编号 | ORD-20260417-001 |
| `customer_id` | text | 客户编号 | CUS-20260417-001 |
| `payment_amount` | number | 收款金额（分） | 113000 |
| `payment_method` | select | 收款方式 | 银行转账/支付宝/微信/现金 |
| `payment_date` | datetime | 收款日期 | Unix 时间戳 |
| `status` | select | 收款状态 | 已收款/已退款 |
| `created_at` | datetime | 创建时间 | Unix 时间戳 |

### 可选字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `payment_account` | text | 收款账户 |
| `transaction_id` | text | 银行流水号 |
| `payer_name` | text | 付款人姓名 |
| `payer_account` | text | 付款人账户 |
| `notes` | text | 备注说明 |

## 收款编号生成规则

格式：`PAY-YYYYMMDD-XXX`

- **日期部分**：`YYYYMMDD`（如 20260417）
- **序号部分**：三位数，从 001 开始
- **重置规则**：每天从 001 重新开始

示例：
- `PAY-20260417-001` - 2026年4月17日的第1笔收款
- `PAY-20260417-002` - 2026年4月17日的第2笔收款

## 收款方式说明

| 方式 | 说明 | 到账时间 |
|------|------|---------|
| **银行转账** | 银行对公/对私转账 | 1-3个工作日 |
| **支付宝** | 支付宝企业账户 | 即时 |
| **微信** | 微信企业账户 | 即时 |
| **现金** | 现金收款 | 即时 |
| **支票** | 银行支票 | 2-5个工作日 |

## 收款状态说明

| 状态 | 说明 |
|------|------|
| **待收款** | 未收到款项 |
| **已收款** | 已收到款项 |
| **部分收款** | 部分收款 |
| **已退款** | 已退款 |

## 输出格式

成功时返回创建的收款记录：

```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "record": {
      "record_id": "rectxxxxxx",
      "fields": {
        "payment_id": "PAY-20260417-001",
        "order_id": "ORD-20260417-001",
        "payment_amount": 113000,
        "payment_method": "银行转账",
        "status": "已收款"
      }
    }
  }
}
```

## 提示

- **获取 table_id**：使用 `lark-cli base tables list --as bot` 查询
- **订单验证**：收款前先确认订单状态为"已发货"
- **金额核对**：核对收款金额与订单金额是否一致
- **凭证保存**：保存银行回单或其他收款凭证
- **财务同步**：收款后应同步更新财务模块

## 业务规则

### 收款条件

- ✅ 订单状态必须为"已发货"或"已开票"
- ✅ 收款金额不能超过订单金额
- ✅ 收款账户信息准确

### 部分收款处理

```
订单总金额：113,000 分（1,130.00 元）

第1次收款：50,000 分（500.00 元）
- 状态：部分收款
- 剩余：63,000 分

第2次收款：63,000 分（630.00 元）
- 状态：已完成
- 剩余：0
```

### 收款确认流程

1. 核实款项到账
2. 确认付款方和金额
3. 记录收款信息
4. 更新订单状态
5. 通知销售和财务

## 错误处理

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| `order not found` | 订单不存在 | 检查 order_id 是否正确 |
| `invalid order status` | 订单状态不允许收款 | 检查订单状态 |
| `amount exceeds order total` | 收款金额超过订单金额 | 检查收款金额 |
| `duplicate payment` | 重复收款 | 检查是否已记录收款 |
| `permission denied` | 权限不足 | 检查 `base:app:write` scope |

## 完整流程

### 场景 1：全额收款

```bash
# 1. 查询待收款订单
ORDER_RESULT=$(lark-cli base app_table_records search \
  --table-id opc_orders \
  --as bot \
  --data '{
    "filter": {
      "conditions": [{
        "field_name": "status",
        "operator": "is",
        "value": ["已发货"]
      }]
    }
  }')

# 2. 提取订单信息
ORDER_ID=$(echo $ORDER_RESULT | jq -r '.data.items[0].fields.order_id')
CUSTOMER_ID=$(echo $ORDER_RESULT | jq -r '.data.items[0].fields.customer_id')
TOTAL_AMOUNT=$(echo $ORDER_RESULT | jq -r '.data.items[0].fields.total_amount')
ORDER_RECORD_ID=$(echo $ORDER_RESULT | jq -r '.data.items[0].record_id')

# 3. 创建收款记录
PAYMENT_DATE=$(date +%Y%m%d)
PAYMENT_ID="PAY-${PAYMENT_DATE}-001"
lark-cli base app_table_records create \
  --table-id opc_payments \
  --as bot \
  --data "{
    \"fields\": {
      \"payment_id\": \"${PAYMENT_ID}\",
      \"order_id\": \"${ORDER_ID}\",
      \"customer_id\": \"${CUSTOMER_ID}\",
      \"payment_amount\": ${TOTAL_AMOUNT},
      \"payment_method\": \"银行转账\",
      \"payment_date\": $(date +%s),
      \"status\": \"已收款\",
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
      "status": "已完成",
      "updated_at": '$(date +%s)'
    }
  }'

# 5. 更新财务记录（应收账款）
# opc-finance +payment-receive
```

### 场景 2：部分收款

```bash
# 第1次收款：部分收款
PAYMENT_AMOUNT=50000

lark-cli base app_table_records create \
  --table-id opc_payments \
  --as bot \
  --data "{
    \"fields\": {
      \"payment_id\": \"PAY-20260417-001\",
      \"order_id\": \"ORD-20260417-001\",
      \"customer_id\": \"CUS-20260417-001\",
      \"payment_amount\": ${PAYMENT_AMOUNT},
      \"payment_method\": \"银行转账\",
      \"payment_date\": $(date +%s),
      \"status\": \"已收款\",
      \"created_at\": $(date +%s)
    }
  }"

# 订单状态保持"已发货"，不更新

# 第2次收款：完成剩余收款
REMAINING_AMOUNT=63000

lark-cli base app_table_records create \
  --table-id opc_payments \
  --as bot \
  --data "{
    \"fields\": {
      \"payment_id\": \"PAY-20260417-002\",
      \"order_id\": \"ORD-20260417-001\",
      \"customer_id\": \"CUS-20260417-001\",
      \"payment_amount\": ${REMAINING_AMOUNT},
      \"payment_method\": \"银行转账\",
      \"payment_date\": $(date +%s),
      \"status\": \"已收款\",
      \"created_at\": $(date +%s)
    }
  }"

# 更新订单状态为"已完成"
lark-cli base app_table_records update \
  --table-id opc_orders \
  --record-id <order_record_id> \
  --as bot \
  --data '{
    "fields": {
      "status": "已完成",
      "updated_at": '$(date +%s)'
    }
  }'
```

### 场景 3：核对收款信息

```bash
# 查询订单和应收记录
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

RECEIVABLE_RESULT=$(lark-cli base app_table_records search \
  --table-id opc_receivables \
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

# 提取应收金额
DUE_AMOUNT=$(echo $RECEIVABLE_RESULT | jq -r '.data.items[0].fields.amount')

# 比对收款金额
if [ $PAYMENT_AMOUNT -eq $DUE_AMOUNT ]; then
  echo "金额匹配，可以确认收款"
else
  echo "金额不匹配，应收：$DUE_AMOUNT，实收：$PAYMENT_AMOUNT"
fi
```

## 后续操作

收款后，可以继续：

1. 财务记账：`opc-finance +payment-receive`
2. 通知客户：发送收款确认通知
3. 更新应收账款：减少应收余额

## 参考

- [order SKILL](../SKILL.md) -- 订单管理全部命令
- [order +query](opc-order-query.md) -- 查询订单
- [order +invoice](opc-order-invoice.md) -- 生成发票
- [finance SKILL](../../opc-finance/SKILL.md) -- 财务管理
- [opc-shared](../../opc-shared/SKILL.md) -- 认证和全局参数
