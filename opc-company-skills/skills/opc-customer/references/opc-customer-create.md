# customer +create

> **前置条件：** 先阅读 [`../../opc-shared/SKILL.md`](../../opc-shared/SKILL.md) 了解认证、全局参数和数据表结构。

创建新客户档案，自动生成客户编号。

需要的scopes: ["base:app:write"]

## 命令

```bash
# 创建客户（交互式）
lark-cli base app_table_records create \
  --table-id <table_id> \
  --as bot \
  --data '{
    "fields": {
      "customer_id": "CUS-20260417-001",
      "company_name": "科技创新有限公司",
      "industry": "互联网",
      "status": "潜在",
      "level": "B",
      "sales_owner": "张三",
      "created_at": 1741564800,
      "updated_at": 1741564800
    }
  }'

# 快捷创建（使用脚本）
CUSTOMER_DATE=$(date +%Y%m%d)
CUSTOMER_ID="CUS-${CUSTOMER_DATE}-001"
lark-cli base app_table_records create \
  --table-id opc_customers \
  --as bot \
  --data "{
    \"fields\": {
      \"customer_id\": \"${CUSTOMER_ID}\",
      \"company_name\": \"示例公司\",
      \"status\": \"潜在\",
      \"level\": \"C\",
      \"created_at\": $(date +%s),
      \"updated_at\": $(date +%s)
    }
  }"
```

## 参数

| 参数 | 必填 | 说明 |
|------|------|------|
| `--table-id <id>` | ✅ | 客户表的 table_id |
| `--as <identity>` | 否 | 身份类型：bot（默认）或 user |
| `--data <json>` | ✅ | 客户数据（JSON 格式） |
| `--dry-run` | 否 | 预览 API 调用，不执行 |

## 数据字段

### 必填字段

| 字段 | 类型 | 说明 | 示例 |
|------|------|------|------|
| `customer_id` | text | 客户编号 | CUS-20260417-001 |
| `company_name` | text | 公司名称 | 科技创新有限公司 |
| `status` | select | 客户状态 | 潜在/意向/成交/流失 |
| `level` | select | 客户等级 | A/B/C |
| `sales_owner` | text | 销售负责人 | 张三 |
| `created_at` | datetime | 创建时间 | Unix 时间戳（秒） |
| `updated_at` | datetime | 更新时间 | Unix 时间戳（秒） |

### 可选字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `industry` | text | 所属行业 |
| `company_size` | text | 公司规模 |
| `address` | text | 公司地址 |
| `phone` | text | 公司电话 |
| `email` | text | 公司邮箱 |
| `website` | text | 公司网址 |

## 客户编号生成规则

格式：`CUS-YYYYMMDD-XXX`

- **日期部分**：`YYYYMMDD`（如 20260417）
- **序号部分**：三位数，从 001 开始
- **重置规则**：每天从 001 重新开始

示例：
- `CUS-20260417-001` - 2026年4月17日的第1个客户
- `CUS-20260417-002` - 2026年4月17日的第2个客户
- `CUS-20260418-001` - 2026年4月18日的第1个客户

## 状态说明

| 状态 | 说明 |
|------|------|
| **潜在** | 初次接触，未明确需求 |
| **意向** | 有明确需求，正在跟进 |
| **成交** | 已签署合同 |
| **流失** | 明确拒绝或长期无响应 |

## 等级说明

| 等级 | 年采购额标准 |
|------|-------------|
| **A** | > 100 万 |
| **B** | 50-100 万 |
| **C** | < 50 万 |

## 输出格式

成功时返回创建的客户记录：

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
        "status": "潜在",
        "level": "B"
      }
    }
  }
}
```

## 提示

- **获取 table_id**：使用 `lark-cli base tables list --as bot` 查询
- **时间戳格式**：使用 `date +%s` 获取当前 Unix 时间戳
- **批量导入**：可结合 CSV 文件批量创建客户
- **编号查重**：创建前建议先查询当天最大编号，避免重复

## 错误处理

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| `table not found` | 表不存在 | 先创建数据表 |
| `customer_id 重复` | 编号冲突 | 查询当天最大编号后 +1 |
| `permission denied` | 权限不足 | 检查 `base:app:write` scope |
| `invalid field` | 字段不存在 | 检查字段名是否正确 |

## 参考

- [customer SKILL](../SKILL.md) -- 客户管理全部命令
- [customer +query](opc-customer-query.md) -- 查询客户
- [customer +contract-create](opc-customer-contract.md) -- 创建合同
- [opc-shared](../../opc-shared/SKILL.md) -- 认证和全局参数
