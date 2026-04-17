# 快速开始指南

本指南将帮助你在 5 分钟内完成 OPC 公司经营管理系统的安装和配置。

## 📋 前置检查

在开始之前，请确保你已具备：

- ✅ Node.js >= 16 已安装
- ✅ npm 或 yarn 已安装
- ✅ 飞书开放平台账号
- ✅ 有权限创建飞书应用的账号

## 🚀 安装步骤

### 步骤 1：安装飞书 CLI（2 分钟）

```bash
# 安装飞书 CLI
npm install -g @larksuite/cli

# 安装 CLI Skills（必需）
npx skills add larksuite/cli -y -g

# 验证安装
lark-cli --version
```

### 步骤 2：创建飞书应用（1 分钟）

1. 访问 [飞书开发者后台](https://open.feishu.cn/app)
2. 点击"创建企业自建应用"
3. 填写应用信息：
   - 应用名称：`OPC 公司管理系统`
   - 应用描述：`基于飞书 CLI 的 ERP 系统`
4. 创建完成，记录 `App ID` 和 `App Secret`

### 步骤 3：配置应用权限（1 分钟）

在飞书开发者后台，为应用开通以下权限：

#### 基础权限（必需）

- `base:app:read` - 读取多维表格
- `base:app:write` - 写入多维表格
- `drive:drive:readonly` - 读取云空间文件

#### 可选权限

- `calendar:calendar:read` - 读取日历
- `calendar:calendar:write` - 写入日历
- `im:message` - 发送消息

### 步骤 4：配置 CLI（1 分钟）

```bash
# 配置应用凭证（交互式）
lark-cli config init

# 按提示输入：
# - App ID: 你记录的 App ID
# - App Secret: 你记录的 App Secret
# - 是否帮助改进 CLI: No
```

### 步骤 5：登录授权（1 分钟）

```bash
# 登录并授权推荐权限
lark-cli auth login --recommend

# 会打开浏览器，完成授权
# 授权完成后返回终端
```

### 步骤 6：验证安装（30 秒）

```bash
# 检查登录状态
lark-cli auth status

# 应显示类似输出：
# ✅ 已登录 as user
# Scopes: calendar:calendar:read, base:app:read, ...
```

## 📊 创建数据表

### 方式 1：手动创建（推荐）

1. 在飞书中创建新的多维表格
2. 命名为 `OPC 公司管理系统`
3. 创建以下数据表：

#### 客户表 (opc_customers)

| 字段名 | 类型 | 说明 |
|--------|------|------|
| customer_id | 文本 | 客户编号 |
| company_name | 文本 | 公司名称 |
| industry | 单选 | 互联网/金融/制造/其他 |
| status | 单选 | 潜在/意向/成交/流失 |
| level | 单选 | A/B/C |
| sales_owner | 文本 | 销售负责人 |
| created_at | 日期时间 | 创建时间 |
| updated_at | 日期时间 | 更新时间 |

#### 订单表 (opc_orders)

| 字段名 | 类型 | 说明 |
|--------|------|------|
| order_id | 文本 | 订单编号 |
| customer_id | 文本 | 客户编号 |
| total_amount | 数字 | 订单金额 |
| status | 单选 | 待处理/已发货/已完成 |
| created_at | 日期时间 | 创建时间 |

### 方式 2：使用模板（即将推出）

```bash
# 即将支持：一键导入完整表结构
lark-cli base template import opc-company-skills
```

## 🎯 第一次使用

### 创建第一个客户

```bash
# 生成客户编号
CUSTOMER_DATE=$(date +%Y%m%d)
CUSTOMER_ID="CUS-${CUSTOMER_DATE}-001"

# 创建客户（需要先获取 table_id）
lark-cli base tables list --as bot

# 记录 opc_customers 表的 table_id，然后：
lark-cli base app_table_records create \
  --table-id <你的table_id> \
  --as bot \
  --data "{
    \"fields\": {
      \"customer_id\": \"${CUSTOMER_ID}\",
      \"company_name\": \"测试客户公司\",
      \"industry\": \"互联网\",
      \"status\": \"潜在\",
      \"level\": \"B\",
      \"sales_owner\": \"你的名字\",
      \"created_at\": $(date +%s),
      \"updated_at\": $(date +%s)
    }
  }"
```

### 查询客户

```bash
lark-cli base app_table_records search \
  --table-id <你的table_id> \
  --as bot \
  --data '{
    "filter": {
      "conditions": [{
        "field_name": "company_name",
        "operator": "contains",
        "value": ["测试"]
      }]
    }
  }'
```

## 🤖 使用 AI Agent

### 安装 Skills

```bash
# 安装 OPC Skills
npx skills add /path/to/opc-company-skills
```

### 与 AI 对话

使用 Claude Code、ChatGPT 等 AI Agent：

```
用户：帮我创建一个新客户，公司叫"科技创新有限公司"

AI：好的，我来帮你创建客户档案。
[调用 opc-customer +create]

✅ 客户创建成功！
- 客户编号：CUS-20260417-001
- 公司名称：科技创新有限公司
- 状态：潜在
- 等级：B
```

## 🐛 常见问题

### 问题 1：权限不足

**错误信息**：`permission_violations`

**解决方案**：
1. 检查飞书开发者后台是否开通权限
2. 确认 `lark-cli auth login` 已完成
3. 尝试重新登录：`lark-cli auth login --recommend`

### 问题 2：表不存在

**错误信息**：`table not found`

**解决方案**：
1. 检查表名是否正确
2. 在飞书中确认表已创建
3. 使用 `lark-cli base tables list` 查看所有表

### 问题 3：认证失败

**错误信息**：`authentication failed`

**解决方案**：
1. 检查 App ID 和 App Secret 是否正确
2. 重新配置：`lark-cli config init`
3. 清除缓存：`rm -rf ~/.lark-cli`

## 📚 下一步

安装完成后，你可以：

1. 📖 阅读 [完整文档](README.md)
2. 🎯 查看 [常见场景](README.md#-常见场景)
3. 🛠️ 了解 [开发指南](README.md#-开发指南)
4. 🤝 加入社区交流

## 💡 提示

- 所有时间戳使用 Unix 时间戳（秒）
- 金额字段使用"分"作为单位（避免浮点误差）
- 状态字段使用预定义的枚举值
- 定期备份数据：`lark-cli base app_table_records export`

---

遇到问题？请提交 [Issue](https://github.com/your-username/opc-company-skills/issues)
