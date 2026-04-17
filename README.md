# awesome-OPC-lark-cli-skills

专为 OPC 定制的 Lark CLI 技能集合，提供高效的命令行工具和自动化工作流。

## 📋 简介

本项目是基于 Lark CLI 的技能集合，专门为 OPC（Operations Center/Platform）团队定制，旨在提供便捷的命令行工具和自动化工作流，提升开发效率和协作体验。

## 📚 项目组成

本项目包含两个主要部分：

### 1. skill-template
Lark CLI 技能开发模板，包含：
- **skill-template.md** - 单模块技能模板
- **master-skill-template.md** - 主技能模板（全功能入口）
- **domains/** - 各业务域的参考文档
  - base.md - 多维表格
  - calendar.md - 日历
  - doc.md - 文档
  - drive.md - 云空间
  - im.md - 消息
  - mail.md - 邮件
  - sheets.md - 电子表格
  - vc.md - 视频会议
  - wiki.md - 知识库

### 2. opc-company-skills
完整的 OPC 公司经营管理系统 ERP 技能套件，包含：
- **opc-shared** - 共享基础（认证、配置、权限管理）
- **opc-customer** - 客户管理（档案、联系人、合同）
- **opc-order** - 订单管理（创建、发货、开票、回款）
- **opc-inventory** - 库存管理（查询、补货、预警）
- **opc-finance** - 财务管理（应收应付、报表）
- **opc-hr** - 人事管理（考勤、薪资、招聘）
- **opc-suite** - 主技能（全功能入口）

## ✨ 特性

- 🚀 **快速部署** - 一键安装和配置
- 🛠️ **定制化工具** - 针对 OPC 工作流优化的技能集
- 📦 **模块化设计** - 灵活的技能组合和管理
- 🔄 **持续更新** - 根据团队需求持续迭代

## 📦 安装

### 前置要求

- Node.js >= 16
- Lark CLI 已安装

### 安装步骤

```bash
# 克隆仓库
git clone https://github.com/cnzhihao/awesome-OPC-lark-cli-skills.git
cd awesome-OPC-lark-cli-skills

# 安装依赖
npm install

# 链接到 Lark CLI
lark skills link
```

## 🎯 使用方法

### 前置要求

- Node.js >= 16
- Lark CLI 已安装
- 飞书开放平台账号

### 安装 Lark CLI

```bash
# 安装 Lark CLI
npm install -g @larksuite/cli

# 安装 CLI Skills
npx skills add larksuite/cli -y -g

# 配置和登录
lark-cli config init
lark-cli auth login --recommend
```

### 使用 OPC 公司经营管理系统

#### 1. 安装

```bash
cd opc-company-skills
npm install
npm run link
```

#### 2. 创建数据表

在飞书中创建多维表格，按照各模块的表结构定义字段。

#### 3. 使用技能

通过 AI Agent（如 Claude Code）自动识别并调用对应 Skill。

**示例对话：**
- 用户：帮我创建一个新客户，公司叫"科技创新有限公司"
- AI：[调用 opc-customer +create]

- 用户：查看今天的库存预警
- AI：[调用 opc-inventory +alert]

- 用户：生成应收账款报表
- AI：[调用 opc-finance +receivable]

### 使用技能模板

#### 1. 查看模板

```bash
# 查看单模块模板
cat skill-template/skill-template.md

# 查看主技能模板
cat skill-template/master-skill-template.md

# 查看业务域参考
cat skill-template/domains/base.md
```

#### 2. 创建新技能

```bash
# 复制模板
cp skill-template/skill-template.md skills/my-skill/SKILL.md

# 根据模板自定义编辑
vim skills/my-skill/SKILL.md
```

### 常用技能

#### skill-template 使用

```bash
# 参考模板创建新技能
cp skill-template/skill-template.md skills/my-new-skill/SKILL.md

# 参考业务域文档
cat skill-template/domains/base.md
```

#### opc-company-skills 使用

```bash
# 进入 opc-company-skills 目录
cd opc-company-skills

# 安装依赖
npm install

# 链接到 Lark CLI
npm run link

# 使用客户管理技能
# AI Agent 会自动识别并调用对应 Skill
# 例如：帮我创建一个新客户，公司叫"科技创新有限公司"
```

## 📁 项目结构

```
awesome-OPC-lark-cli-skills/
├── skill-template/              # 技能开发模板
│   ├── skill-template.md       # 单模块模板
│   ├── master-skill-template.md # 主技能模板
│   └── domains/                # 业务域参考文档
│       ├── base.md             # 多维表格
│       ├── calendar.md         # 日历
│       ├── doc.md              # 文档
│       ├── drive.md            # 云空间
│       ├── im.md               # 消息
│       ├── mail.md             # 邮件
│       ├── sheets.md           # 电子表格
│       ├── vc.md               # 视频会议
│       └── wiki.md             # 知识库
│
├── opc-company-skills/         # OPC 公司经营管理系统
│   ├── README.md               # 详细说明
│   ├── QUICKSTART.md           # 快速开始
│   ├── PROJECT_STRUCTURE.md    # 项目结构
│   ├── package.json            # npm 配置
│   └── skills/                 # 技能模块
│       ├── opc-shared/         # 共享基础
│       ├── opc-customer/       # 客户管理
│       ├── opc-order/          # 订单管理
│       ├── opc-inventory/      # 库存管理
│       ├── opc-finance/        # 财务管理
│       ├── opc-hr/             # 人事管理
│       └── opc-suite/          # 主技能
│
├── .gitignore                  # Git 忽略配置
└── README.md                   # 项目说明（本文件）
```

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 开启 Pull Request

## 📄 许可证

本项目采用 MIT 许可证 - 详见 [LICENSE](LICENSE) 文件

## 👥 作者

- **cnzhihao** - *初始工作*

## 🙏 致谢

- Lark CLI 团队提供的优秀框架
- OPC 团队的支持和反馈

## 📮 联系方式

如有问题或建议，请通过以下方式联系：

- 提交 [Issue](https://github.com/cnzhihao/awesome-OPC-lark-cli-skills/issues)
- 邮箱：your-email@example.com

---

⭐ 如果这个项目对你有帮助，请给它一个星标！
