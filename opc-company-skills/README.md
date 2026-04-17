# OPC 公司经营管理系统

> 基于飞书 CLI 构建的企业资源规划（ERP）系统 AI Agent Skills 套件

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Lark CLI](https://img.shields.io/badge/lark--cli-v1.0.14+-blue.svg)](https://github.com/larksuite/cli)

## 📋 项目简介

OPC 公司经营管理系统是一套完整的 AI Agent Skills，通过飞书 CLI 操作飞书套件（多维表格、日历、文档等），实现企业的全流程数字化管理。

### 核心特性

- 🎯 **全业务覆盖**：客户、订单、库存、财务、人事五大核心模块
- 🤖 **AI 原生设计**：专为 AI Agent 优化，结构化指令，高成功率
- 📊 **数据可视化**：基于飞书多维表格，支持仪表盘、图表、透视表
- 🔒 **权限安全**：多级权限控制，操作审计，数据隔离
- 🚀 **开箱即用**：3 分钟完成配置，即刻开始使用

## 🏗️ 系统架构

```
飞书开放平台
├── 多维表格 (Base)     # 业务数据存储
├── 日历 (Calendar)     # 日程管理、提醒
├── 文档 (Docs)         # 合同、报告模板
└── 消息 (IM)          # 通知、协作

          ↑
          │ lark-cli
          │
OPC AI Agent Skills
├── opc-shared       # 共享基础（认证、配置）
├── opc-customer     # 客户管理
├── opc-order        # 订单管理
├── opc-inventory    # 库存管理
├── opc-finance      # 财务管理
└── opc-hr           # 人事管理
```

## 📦 模块说明

| 模块 | 功能 | 状态 |
|------|------|------|
| **opc-shared** | 认证、配置、权限管理 | ✅ 完成 |
| **opc-customer** | 客户档案、联系人、合同管理 | ✅ 完成 |
| **opc-order** | 订单创建、发货、开票、回款 | 🚧 开发中 |
| **opc-inventory** | 库存查询、补货、预警 | 🚧 开发中 |
| **opc-finance** | 应收应付、财务报表 | 🚧 开发中 |
| **opc-hr** | 考勤、薪资、招聘 | 🚧 开发中 |

## 🚀 快速开始

### 前置要求

- Node.js >= 16
- 飞书开放平台账号
- 飞书 CLI 已安装

### 安装步骤

#### 1. 安装飞书 CLI

```bash
npm install -g @larksuite/cli

# 安装 CLI Skills
npx skills add larksuite/cli -y -g
```

#### 2. 配置应用凭证

```bash
# 创建飞书应用（在飞书开发者后台）
# 获取 App ID 和 App Secret

# 配置 CLI
lark-cli config init
```

#### 3. 登录授权

```bash
# 推荐使用自动选择的常用权限
lark-cli auth login --recommend
```

#### 4. 验证安装

```bash
# 检查登录状态
lark-cli auth status

# 查看可用命令
lark-cli --help
```

### 创建数据表

在飞书中创建多维表格，按以下结构创建数据表：

#### 客户表 (opc_customers)

| 字段名 | 类型 | 说明 |
|--------|------|------|
| customer_id | 文本 | 客户编号 |
| company_name | 文本 | 公司名称 |
| industry | 文本 | 所属行业 |
| status | 单选 | 客户状态 |
| level | 单选 | 客户等级 |
| sales_owner | 文本 | 销售负责人 |
| created_at | 日期 | 创建时间 |

详细表结构见各模块的 `SKILL.md` 文档。

## 💻 使用示例

### 客户管理

```bash
# 创建新客户
lark-cli base app_table_records create \
  --table-id opc_customers \
  --as bot \
  --data '{
    "fields": {
      "customer_id": "CUS-20260417-001",
      "company_name": "科技创新有限公司",
      "industry": "互联网",
      "status": "潜在",
      "level": "B"
    }
  }'

# 查询客户
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
```

### AI Agent 使用

使用 Claude Code、ChatGPT 等 AI Agent 时，直接加载 Skills：

```bash
# 安装 Skills
npx skills add /path/to/opc-company-skills

# AI Agent 会自动识别并调用对应 Skill
# 示例对话：
用户：帮我创建一个新客户，公司叫"科技创新有限公司"
AI：[调用 opc-customer +create]
```

## 📚 文档结构

```
opc-company-skills/
├── README.md                    # 项目总览（本文件）
├── package.json                 # npm 包配置
├── skills/                      # Skills 目录
│   ├── opc-shared/             # 共享基础
│   │   └── SKILL.md
│   ├── opc-customer/           # 客户管理
│   │   ├── SKILL.md            # 主文档
│   │   └── references/         # 详细操作指南
│   │       ├── opc-customer-create.md
│   │       ├── opc-customer-query.md
│   │       └── opc-customer-contract.md
│   ├── opc-order/              # 订单管理
│   ├── opc-inventory/          # 库存管理
│   ├── opc-finance/            # 财务管理
│   ├── opc-hr/                 # 人事管理
│   └── opc-suite/              # 主 Skill（全功能入口）
│       └── SKILL.md
└── docs/                        # 额外文档
    ├── ARCHITECTURE.md          # 系统架构
    └── API.md                   # API 参考
```

## 🎯 常见场景

### 场景 1：新客户下单流程

1. 创建客户档案 → `opc-customer +create`
2. 创建销售订单 → `opc-order +create`
3. 检查库存 → `opc-inventory +check`
4. 生成发货单 → `opc-order +ship`
5. 生成发票 → `opc-order +invoice`
6. 记录收款 → `opc-finance +payment-receive`

### 场景 2：库存预警处理

1. 查看库存预警 → `opc-inventory +alert`
2. 创建补货单 → `opc-inventory +restock`
3. 联系供应商 → 发送飞书消息
4. 更新库存 → `opc-inventory +update`

### 场景 3：财务月结对账

1. 查询应收账款 → `opc-finance +receivable`
2. 查询应付账款 → `opc-finance +payable`
3. 生成财务报表 → `opc-finance +report`
4. 导出数据 → `lark-cli base app_table_records export`

## 🔒 权限与安全

### 基础权限

所有操作需要以下飞书开放平台权限：

- `base:app:read` - 读取多维表格
- `base:app:write` - 写入多维表格

### 特殊权限

- 财务模块需要额外审批流程
- 删除操作需要管理员权限
- 敏感数据仅授权人员可见

### 安全措施

- 所有操作记录审计日志
- 敏感操作需要二次确认
- 数据权限按角色隔离
- 支持操作回滚

## 🛠️ 开发指南

### 添加新模块

1. 在 `skills/` 目录创建新模块文件夹
2. 创建 `SKILL.md` 文件
3. 创建 `references/` 目录和详细文档
4. 在 `opc-suite/SKILL.md` 中添加模块说明

### 参考模板

本项目基于飞书 CLI 的官方模板开发：

- `skill-template/skill-template.md` - 单模块模板
- `skill-template/master-skill-template.md` - 主 Skill 模板

### 贡献指南

1. Fork 本项目
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 提交 Pull Request

## 📖 相关资源

- [飞书开放平台](https://open.feishu.cn/)
- [飞书 CLI 文档](https://github.com/larksuite/cli)
- [飞书多维表格 API](https://open.feishu.cn/document/server-docs/docs/bitable-v1/app-table-list)
- [AI Agent Skills 开发指南](https://github.com/larksuite/cli/blob/main/skills/README.md)

## 🏆 比赛信息

本项目参与 **[比赛名称]**，作品类别：飞书 CLI Skill 开发。

### 作品亮点

- ✅ 完整的 ERP 系统设计，覆盖企业核心业务流程
- ✅ 基于 AI Agent 原生设计，结构化指令，高可维护性
- ✅ 开箱即用，3 分钟完成配置
- ✅ 详细的文档和使用示例
- ✅ 模块化设计，易于扩展

## 📝 许可证

本项目采用 MIT 许可证 - 详见 [LICENSE](LICENSE) 文件

## 🤝 联系方式

- 作者：[你的名字]
- 邮箱：[your-email@example.com]
- GitHub：[your-username]
- 项目链接：[https://github.com/your-username/opc-company-skills](https://github.com/your-username/opc-company-skills)

---

⭐ 如果这个项目对你有帮助，请给它一个 Star！
