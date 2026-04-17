# customer +contract-create

> **前置条件：** 先阅读 [`../../opc-shared/SKILL.md`](../../opc-shared/SKILL.md) 了解认证、全局参数和数据表结构。

为客户创建合同记录。

需要的scopes: ["base:app:write"]

## 命令

```bash
# 创建合同
lark-cli base app_table_records create \
  --table-id opc_contracts \
  --as bot \
  --data '{
    "fields": {
      "contract_id": "CON-20260417-001",
      "customer_id": "CUS-20260417-001",
      "contract_name": "年度采购合同",
      "contract_type": "采购合同",
      "amount": 1000000,
      "start_date": 1741564800,
      "end_date": 1773100799,
      "status": "执行中",
      "sales_owner": "张三",
      "created_at": 1741564800
    }
  }'

# 快捷创建（使用脚本）
CONTRACT_DATE=$(date +%Y%m%d)
CONTRACT_ID="CON-${CONTRACT_DATE}-001"
lark-cli base app_table_records create \
  --table-id opc_contracts \
  --as bot \
  --data "{
    \"fields\": {
      \"contract_id\": \"${CONTRACT_ID}\",
      \"customer_id\": \"${CUSTOMER_ID}\",
      \"contract_name\": \"${CONTRACT_NAME}\",
      \"contract_type\": \"采购合同\",
      \"amount\": ${AMOUNT},
      \"start_date\": $(date +%s),
      \"end_date\": $(date -d '+1 year' +%s),
      \"status\": \"执行中\",
      \"sales_owner\": \"${SALES_OWNER}\",
      \"created_at\": $(date +%s)
    }
  }"
```

## 参数

| 参数 | 必填 | 说明 |
|------|------|------|
| `--table-id <id>` | ✅ | 合同表的 table_id |
| `--as <identity>` | 否 | 身份类型：bot（默认）或 user |
| `--data <json>` | ✅ | 合同数据（JSON 格式） |
| `--dry-run` | 否 | 预览 API 调用，不执行 |

## 数据字段

### 必填字段

| 字段 | 类型 | 说明 | 示例 |
|------|------|------|------|
| `contract_id` | text | 合同编号 | CON-20260417-001 |
| `customer_id` | text | 客户编号 | CUS-20260417-001 |
| `contract_name` | text | 合同名称 | 年度采购合同 |
| `contract_type` | select | 合同类型 | 采购合同/服务合同/框架协议 |
| `amount` | number | 合同金额（分） | 1000000（10000.00元） |
| `start_date` | datetime | 合同开始日期 | Unix 时间戳 |
| `end_date` | datetime | 合同结束日期 | Unix 时间戳 |
| `status` | select | 合同状态 | 草稿/审批中/执行中/已完成/已终止 |
| `sales_owner` | text | 销售负责人 | 张三 |
| `created_at` | datetime | 创建时间 | Unix 时间戳 |

### 可选字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `payment_terms` | text | 付款条件 |
| `delivery_terms` | text | 交付条件 |
| `notes` | text | 备注说明 |
| `attachment_url` | url | 合同附件链接 |

## 合同编号生成规则

格式：`CON-YYYYMMDD-XXX`

- **日期部分**：`YYYYMMDD`（如 20260417）
- **序号部分**：三位数，从 001 开始
- **重置规则**：每天从 001 重新开始

示例：
- `CON-20260417-001` - 2026年4月17日的第1个合同
- `CON-20260417-002` - 2026年4月17日的第2个合同

## 合同类型说明

| 类型 | 说明 | 适用场景 |
|------|------|---------|
| **采购合同** | 产品采购 | 客户购买产品 |
| **服务合同** | 服务提供 | 客户购买服务 |
| **框架协议** | 长期合作 | 确立长期合作关系 |

## 合同状态流转

```
草稿 → 审批中 → 执行中 → 已完成
          ↓          ↓
        已终止    已终止
```

| 状态 | 说明 | 可执行操作 |
|------|------|-----------|
| **草稿** | 合同创建中 | 编辑、提交审批 |
| **审批中** | 合同审批流程中 | 审批通过/拒绝 |
| **执行中** | 合同执行中 | 终止、完成 |
| **已完成** | 合同完成 | 归档 |
| **已终止** | 合同终止 | - |

## 输出格式

成功时返回创建的合同记录：

```json
{
  "code": 0,
  "msg": "success",
  "data": {
    "record": {
      "record_id": "rectxxxxxx",
      "fields": {
        "contract_id": "CON-20260417-001",
        "customer_id": "CUS-20260417-001",
        "contract_name": "年度采购合同",
        "amount": 1000000,
        "status": "审批中"
      }
    }
  }
}
```

## 提示

- **获取 table_id**：使用 `lark-cli base tables list --as bot` 查询
- **客户验证**：创建前建议先验证客户是否存在
- **金额单位**：合同金额单位为分（避免浮点误差）
- **日期计算**：使用 `date -d '+1 year' +%s` 计算一年后的时间戳
- **合同附件**：可上传合同扫描件到飞书云空间，然后在附件字段中添加链接

## 业务规则

### 合同金额计算

- **单位**：分（避免浮点误差）
- **示例**：10,000.00 元 = 1,000,000 分

### 合同期限

- **短期合同**：< 1年
- **中期合同**：1-3年
- **长期合同**：> 3年

### 合同审批

- **金额 < 10万**：销售主管审批
- **金额 10-50万**：部门经理审批
- **金额 > 50万**：总经理审批

## 错误处理

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| `customer not found` | 客户不存在 | 先创建客户档案 |
| `table not found` | 表不存在 | 先创建数据表 |
| `permission denied` | 权限不足 | 检查 `base:app:write` scope |
| `invalid amount` | 金额格式错误 | 确保金额为整数（分） |
| `invalid date range` | 日期范围错误 | 确保结束日期大于开始日期 |

## 常见场景

### 场景 1：新客户首单合同

```bash
# 1. 创建客户档案
# customer +create

# 2. 创建首单合同
lark-cli base app_table_records create \
  --table-id opc_contracts \
  --as bot \
  --data '{
    "fields": {
      "contract_id": "CON-20260417-001",
      "customer_id": "CUS-20260417-001",
      "contract_name": "首单采购合同",
      "contract_type": "采购合同",
      "amount": 100000,
      "start_date": 1741564800,
      "end_date": 1773100799,
      "status": "审批中",
      "created_at": 1741564800
    }
  }'
```

### 场景 2：老客户续约

```bash
# 1. 查询客户历史合同
# customer +contract-query

# 2. 创建续约合同
lark-cli base app_table_records create \
  --table-id opc_contracts \
  --as bot \
  --data '{
    "fields": {
      "contract_id": "CON-20260417-002",
      "customer_id": "CUS-20260417-001",
      "contract_name": "续约合同",
      "contract_type": "框架协议",
      "amount": 2000000,
      "start_date": 1773100800,
      "end_date": 1804636799,
      "status": "审批中",
      "notes": "续约第二年，金额翻倍",
      "created_at": 1741564800
    }
  }'
```

## 参考

- [customer SKILL](../SKILL.md) -- 客户管理全部命令
- [customer +query](opc-customer-query.md) -- 查询客户
- [customer +create](opc-customer-create.md) -- 创建客户
- [order SKILL](../../opc-order/SKILL.md) -- 订单管理
- [opc-shared](../../opc-shared/SKILL.md) -- 认证和全局参数
