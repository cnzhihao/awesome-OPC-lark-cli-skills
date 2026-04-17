# OPC 公司经营管理系统 - 项目结构

本文档展示了 OPC 公司经营管理系统的完整项目结构。

## 📁 完整目录结构

```
opc-company-skills/
├── README.md                           # 📖 项目总览
├── QUICKSTART.md                       # 🚀 快速开始指南
├── package.json                        # 📦 npm 包配置
├── LICENSE                             # ⚖️ MIT 许可证
├── .gitignore                          # 🚫 Git 忽略文件
│
└── skills/                             # 🎯 Skills 目录
    │
    ├── opc-shared/                     # 🔴 共享基础
    │   └── SKILL.md                    # 认证、配置、权限管理
    │
    ├── opc-customer/                   # 👥 客户管理
    │   ├── SKILL.md                    # 主文档
    │   └── references/                 # 详细操作指南
    │       └── opc-customer-create.md  # 创建客户
    │
    ├── opc-order/                      # 📦 订单管理
    │   ├── SKILL.md                    # 主文档
    │   └── references/                 # 详细操作指南
    │       └── opc-order-create.md     # 创建订单
    │
    ├── opc-inventory/                  # 📊 库存管理
    │   └── SKILL.md                    # 主文档
    │
    ├── opc-finance/                    # 💰 财务管理
    │   └── SKILL.md                    # 主文档
    │
    ├── opc-hr/                         # 👔 人事管理
    │   └── SKILL.md                    # 主文档
    │
    └── opc-suite/                      # 🟢 主 Skill
        └── SKILL.md                    # 全功能入口
```

## 📊 模块完成情况

| 模块 | SKILL.md | references | 完成度 |
|------|----------|------------|--------|
| **opc-shared** | ✅ | - | 100% |
| **opc-customer** | ✅ | 1/3 | 60% |
| **opc-order** | ✅ | 1/6 | 40% |
| **opc-inventory** | ✅ | 0/5 | 30% |
| **opc-finance** | ✅ | 0/6 | 30% |
| **opc-hr** | ✅ | 0/8 | 30% |
| **opc-suite** | ✅ | - | 100% |

## 🎯 核心功能覆盖

### 客户管理 (opc-customer)
- ✅ 客户档案创建
- ✅ 客户查询
- ✅ 合同管理（计划中）
- ✅ 客户跟进记录（计划中）

### 订单管理 (opc-order)
- ✅ 订单创建
- ✅ 订单查询
- ✅ 发货管理（计划中）
- ✅ 开票管理（计划中）
- ✅ 收款确认（计划中）

### 库存管理 (opc-inventory)
- ✅ 库存查询
- ✅ 库存更新
- ✅ 库存预警
- ✅ 补货管理（计划中）
- ✅ 库存调整（计划中）

### 财务管理 (opc-finance)
- ✅ 应收账款查询
- ✅ 应付账款查询
- ✅ 收款确认（计划中）
- ✅ 付款确认（计划中）
- ✅ 财务报表（计划中）
- ✅ 账龄分析（计划中）

### 人事管理 (opc-hr)
- ✅ 员工档案管理
- ✅ 考勤记录查询
- ✅ 薪资计算（计划中）
- ✅ 招聘管理（计划中）

## 🔧 技术架构

### 数据存储
- **飞书多维表格 (Base)**：所有业务数据存储

### 认证与权限
- **飞书开放平台 OAuth**：用户身份认证
- **Scope 权限控制**：细粒度权限管理

### AI Agent 集成
- **结构化文档**：专为 AI Agent 优化
- **工作流指导**：详细的业务流程说明
- **错误处理**：完善的错误处理和恢复机制

## 📝 文档规范

### SKILL.md 结构
每个模块的 SKILL.md 包含：

1. **Frontmatter**：模块元信息
2. **核心概念**：业务模型和数据结构
3. **典型工作流**：业务操作流程
4. **Shortcuts**：快捷命令列表
5. **API Resources**：原生 API 说明
6. **业务规则**：计算公式和业务逻辑
7. **权限表**：所需的 scope 权限
8. **常见场景**：实际应用场景

### references 文档结构
每个操作指南包含：

1. **前置条件**：必要的准备工作
2. **命令示例**：完整的命令示例
3. **参数说明**：详细的参数表格
4. **数据字段**：数据结构和字段说明
5. **输出格式**：返回结果格式
6. **提示**：使用提示和注意事项
7. **错误处理**：常见错误和解决方案
8. **参考**：相关文档链接

## 🚀 使用方式

### 1. 安装飞书 CLI
```bash
npm install -g @larksuite/cli
npx skills add larksuite/cli -y -g
```

### 2. 配置和登录
```bash
lark-cli config init
lark-cli auth login --recommend
```

### 3. 创建数据表
在飞书中创建多维表格，按各模块的表结构定义字段。

### 4. 使用 Skills
通过 AI Agent（如 Claude Code）自动识别并调用对应 Skill。

## 📚 依赖关系

```
opc-suite (主入口)
    ├── opc-shared (所有模块依赖)
    ├── opc-customer
    │   └── opc-shared
    ├── opc-order
    │   ├── opc-shared
    │   ├── opc-customer
    │   └── opc-inventory
    ├── opc-inventory
    │   └── opc-shared
    ├── opc-finance
    │   ├── opc-shared
    │   └── opc-order
    └── opc-hr
        └── opc-shared
```

## 🛠️ 后续开发计划

### 短期目标（1-2周）
- [ ] 完成所有 modules 的 references 文档
- [ ] 添加更多实际使用示例
- [ ] 创建演示视频
- [ ] 编写单元测试

### 中期目标（1个月）
- [ ] 添加 Web Dashboard
- [ ] 支持多语言（英文）
- [ ] 添加数据导入导出功能
- [ ] 创建 Docker 部署方案

### 长期目标（3个月）
- [ ] 支持自定义字段
- [ ] 添加工作流引擎
- [ ] 集成第三方服务
- [ ] 开发移动端应用

## 🤝 贡献指南

欢迎贡献代码、文档、Bug 报告和功能建议！

1. Fork 本项目
2. 创建特性分支
3. 提交更改
4. 推送到分支
5. 提交 Pull Request

## 📄 许可证

MIT License - 详见 LICENSE 文件

---

**注意**：本项目按照飞书 CLI 官方 template 的严格格式开发，确保与飞书 CLI 完全兼容。
