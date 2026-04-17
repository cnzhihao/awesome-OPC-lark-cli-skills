# order +invoice

> **前置条件：** 先阅读 [`../../opc-shared/SKILL.md`](../../opc-shared/SKILL.md) 了解认证、全局参数和数据表结构。

生成发票，为已发货订单开具发票。

需要的scopes: ["base:app:write"]

## 命令

```bash
# 创建发票
lark-cli base app_table_records create \
  --table-id opc_invoices \
  --as bot \
  --data '{
    "fields": {
      "invoice_id": "INV-20260417-001",
      "order_id": "ORD-20260417-001",
      "customer_id": "CUS-20260417-001",
      "invoice_type": "增值税普通发票",
      "invoice_title": "科技创新有限公司",
      "tax_number": "91110000XXXXXXXX",
      "amount": 100000,
      "tax_amount": 13000,
      "total_amount": 113000,
      "invoice_date": 1741564800,
      "status": "已开具",
      "created_at": 1741564800
    }
  }'

# 快捷开票（使用脚本）
INVOICE_DATE=$(date +%Y%m%d)
INVOICE_ID="INV-${INVOICE_DATE}-001"
lark-cli base app_table_records create \
  --table-id opc_invoices \
  --as bot \
  --data "{
    \"fields\": {
      \"invoice_id\": \"${INVOICE_ID}\",
      \"order_id\": \"${ORDER_ID}\",
      \"customer_id\": \"${CUSTOMER_ID}\",
      \"invoice_type\": \"增值税普通发票\",
      \"invoice_title\": \"${COMPANY_NAME}\",
      \"tax_number\": \"${TAX_NUMBER}\",
      \"amount\": ${AMOUNT},
      \"tax_amount\": ${TAX_AMOUNT},
      \"total_amount\": ${TOTAL_AMOUNT},
      \"invoice_date\": $(date +%s),
      \"status\": \"已开具\",
      \"created_at\": $(date +%s)
    }
  }"
```

## 参数

| 参数 | 必填 | 说明 |
|------|------|------|
| `--table-id <id>` | ✅ | 发票表的 table_id |
| `--as <identity>` | 否 | 身份类型：bot（默认）或 user |
| `--data <json>` | ✅ | 发票数据（JSON 格式） |
| `--dry-run` | 否 | 预览 API 调用，不执行 |

## 数据字段

### 必填字段

| 字段 | 类型 | 说明 | 示例 |
|------|------|------|------|
| `invoice_id` | text | 发票号码 | INV-20260417-001 |
| `order_id` | text | 订单编号 | ORD-20260417-001 |
| `customer_id` | text | 客户编号 | CUS-20260417-001 |
| `invoice_type` | select | 发票类型 | 增值税普通发票/增值税专用发票 |
| `invoice_title` | text | 发票抬头 | 科技创新有限公司 |
| `tax_number` | text | 税号 | 91110000XXXXXXXX |
| `amount` | number | 金额（分） | 100000 |
| `tax_amount` | number | 税额（分） | 13000 |
| `total_amount` | number | 价税合计（分） | 113000 |
| `invoice_date` | datetime | 开票日期 | Unix 时间戳 |
| `status` | select | 发票状态 | 已开具/已发送/已作废 |
| `created_at` | datetime | 创建时间 | Unix 时间戳 |

### 可选字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `invoice_address` | text | 发票寄送地址 |
| `invoice_phone` | text | 联系电话 |
| `bank_name` | text | 开户行 |
| `bank_account` | text | 银行账号 |
| `notes` | text | 备注说明 |
| `attachment_url` | url | 发票扫描件链接 |

## 发票编号生成规则

格式：`INV-YYYYMMDD-XXX`

- **日期部分**：`YYYYMMDD`（如 20260417）
- **序号部分**：三位数，从 001 开始
- **重置规则**：每天从 001 重新开始

示例：
- `INV-20260417-001` - 2026年4月17日的第1张发票
- `INV-20260417-002` - 2026年4月17日的第2张发票

## 发票类型说明

| 类型 | 税率 | 适用场景 |
|------|------|---------|
| **增值税普通发票** | 13% | 一般客户 |
| **增值税专用发票** | 13% | 企业客户（可抵扣） |
| **免税发票** | 0% | 出口或免税商品 |

## 发票状态说明

| 状态 | 说明 |
|------|------|
| **已开具** | 发票已开具 |
| **已发送** | 发票已发送给客户 |
| **已作废** | 发票已作废 |
| **已红冲** | 发票已红冲 |

## 税额计算

### 增值税计算（税率13%）

```
税额 = 金额 × 税率
价税合计 = 金额 + 税额

示例：
- 金额：100,000 分（1,000.00 元）
- 税额：100,000 × 13% = 13,000 分（130.00 元）
- 价税合计：100,000 + 13,000 = 113,000 分（1,130.00 元）
```

### 税率表

| 税率 | 适用商品/服务 |
|------|-------------|
| 13% | 一般商品、加工修理修配劳务 |
| 9% | 交通运输、邮政、基础电信、建筑、不动产 |
| 6% | 现代服务、生活服务、无形资产 |
| 0% | 出口货物、境内单位和个人发生的跨境应税行为 |

## 输出格式

成功时返回创建的发票记录：

```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "record": {
      "record_id": "rectxxxxxx",
      "fields": {
        "invoice_id": "INV-20260417-001",
        "order_id": "ORD-20260417-001",
        "invoice_type": "增值税普通发票",
        "amount": 100000,
        "tax_amount": 13000,
        "total_amount": 113000,
        "status": "已开具"
      }
    }
  }
}
```

## 提示

- **获取 table_id**：使用 `lark-cli base tables list --as bot` 查询
- **订单验证**：开票前先确认订单状态为"已发货"
- **金额核对**：核对发票金额与订单金额是否一致
- **税号验证**：开具专用发票前必须验证税号
- **发票附件**：上传发票扫描件到飞书云空间

## 业务规则

### 开票条件

- ✅ 订单状态必须为"已发货"
- ✅ 客户信息完整（抬头、税号）
- ✅ 发票金额与订单金额一致
- ✅ 开票信息准确无误

### 发票金额计算

```
发票金额 = 订单总金额
税额 = 发票金额 × 税率
价税合计 = 发票金额 + 税额
```

### 发票类型选择

| 客户类型 | 推荐发票类型 |
|---------|------------|
| 个人 | 增值税普通发票 |
| 企业（小规模） | 增值税普通发票 |
| 企业（一般纳税人） | 增值税专用发票 |

## 错误处理

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| `order not found` | 订单不存在 | 检查 order_id 是否正确 |
| `invalid order status` | 订单状态不允许开票 | 只有"已发货"状态可以开票 |
| `invalid tax number` | 税号格式错误 | 检查税号是否正确（18位） |
| `amount mismatch` | 金额不匹配 | 检查发票金额与订单金额是否一致 |
| `permission denied` | 权限不足 | 检查 `base:app:write` scope |

## 完整流程

### 场景 1：普通发票开具

```bash
# 1. 查询已发货订单
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

# 3. 计算税额
AMOUNT=$TOTAL_AMOUNT
TAX_RATE=0.13
TAX_AMOUNT=$(echo "$AMOUNT * $TAX_RATE" | bc | cut -d'.' -f1)
TOTAL_AMOUNT_WITH_TAX=$(echo "$AMOUNT + $TAX_AMOUNT" | bc | cut -d'.' -f1)

# 4. 查询客户信息
CUSTOMER_RESULT=$(lark-cli base app_table_records search \
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

COMPANY_NAME=$(echo $CUSTOMER_RESULT | jq -r '.data.items[0].fields.company_name')

# 5. 创建发票
INVOICE_DATE=$(date +%Y%m%d)
INVOICE_ID="INV-${INVOICE_DATE}-001"
lark-cli base app_table_records create \
  --table-id opc_invoices \
  --as bot \
  --data "{
    \"fields\": {
      \"invoice_id\": \"${INVOICE_ID}\",
      \"order_id\": \"${ORDER_ID}\",
      \"customer_id\": \"${CUSTOMER_ID}\",
      \"invoice_type\": \"增值税普通发票\",
      \"invoice_title\": \"${COMPANY_NAME}\",
      \"amount\": ${AMOUNT},
      \"tax_amount\": ${TAX_AMOUNT},
      \"total_amount\": ${TOTAL_AMOUNT_WITH_TAX},
      \"invoice_date\": $(date +%s),
      \"status\": \"已开具\",
      \"created_at\": $(date +%s)
    }
  }"
```

### 场景 2：专用发票开具

```bash
# 专用发票需要更多信息
lark-cli base app_table_records create \
  --table-id opc_invoices \
  --as bot \
  --data '{
    "fields": {
      "invoice_id": "INV-20260417-002",
      "order_id": "ORD-20260417-001",
      "customer_id": "CUS-20260417-001",
      "invoice_type": "增值税专用发票",
      "invoice_title": "科技创新有限公司",
      "tax_number": "91110000XXXXXXXX",
      "amount": 100000,
      "tax_amount": 13000,
      "total_amount": 113000,
      "invoice_address": "北京市朝阳区某某街道",
      "invoice_phone": "010-12345678",
      "bank_name": "中国工商银行北京分行",
      "bank_account": "1234567890",
      "invoice_date": 1741564800,
      "status": "已开具",
      "created_at": 1741564800
    }
  }'
```

## 后续操作

开票后，可以继续：

1. 发送发票：寄送纸质发票或发送电子发票
2. 确认收款：`opc-order +payment-receive`
3. 财务记账：记录应收账款

## 参考

- [order SKILL](../SKILL.md) -- 订单管理全部命令
- [order +query](opc-order-query.md) -- 查询订单
- [order +ship](opc-order-ship.md) -- 创建发货单
- [order +payment-receive](opc-order-payment.md) -- 确认收款
- [finance SKILL](../../opc-finance/SKILL.md) -- 财务管理
- [opc-shared](../../opc-shared/SKILL.md) -- 认证和全局参数
