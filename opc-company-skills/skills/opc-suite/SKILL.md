---
name: opc-suite
version: 1.0.0
description: "OPC 公司经营管理系统 - 全功能套件：基于飞书 CLI 的企业资源规划系统，涵盖客户管理、订单管理、库存管理、财务管理、人事管理等核心业务领域。"
metadata:
  category: "erp"
  requires:
    bins: ["lark-cli"]
---

# OPC 公司经营管理系统

你是 AI Agent，通过 lark-cli 命令和飞书套件操作 OPC 公司经营管理系统。

**CRITICAL — 开始前 MUST 先用 Read 工具读取 [`opc-shared/SKILL.md`](opc-shared/SKILL.md)，其中包含认证、权限处理、数据表结构**

## 系统概述

OPC 公司经营管理系统是基于飞书套件构建的**一体化企业资源规划（ERP）系统**，通过飞书多维表格存储业务数据，通过 lark-cli 执行所有操作。

### 核心模块

| 模块 | 功能 | 文档 |
|------|------|------|
| **客户管理** | 客户档案、联系人、合同管理 | `opc-customer/SKILL.md` |
| **订单管理** | 订单创建、发货、开票、回款 | `opc-order/SKILL.md` |
| **库存管理** | 库存查询、补货、预警 | `opc-inventory/SKILL.md` |
| **财务管理** | 应收应付、财务报表、成本分析 | `opc-finance/SKILL.md` |
| **人事管理** | 考勤、薪资、招聘 | `opc-hr/SKILL.md` |

### 数据架构

```
飞书多维表格 (Base)
├── opc_customers       # 客户表
├── opc_contacts        # 联系人表
├── opc_contracts       # 合同表
├── opc_orders          # 订单表
├── opc_order_items     # 订单明细表
├── opc_products        # 产品表
├── opc_inventory       # 库存表
├── opc_invoices        # 发票表
├── opc_payments        # 收付款表
├── opc_employees       # 员工表
└── opc_attendance      # 考勤表
```

## 快速开始

### 首次使用

```bash
# 1. 安装飞书 CLI
npm install -g @larksuite/cli

# 2. 配置应用
lark-cli config init

# 3. 登录授权
lark-cli auth login --recommend

# 4. 验证登录状态
lark-cli auth status
```

### 创建数据表

```bash
# 在飞书中创建多维表格
# 表结构见各模块的 SKILL.md 文档
```

## 常见场景

### 场景 1：新客户下单

```
1. 创建客户档案
   → opc-customer +create

2. 创建销售订单
   → opc-order +create

3. 检查库存
   → opc-inventory +check

4. 生成发货单
   → opc-order +ship

5. 生成发票
   → opc-order +invoice

6. 记录收款
   → opc-finance +payment-receive
```

### 场景 2：库存预警处理

```
1. 查看库存预警
   → opc-inventory +alert

2. 创建补货单
   → opc-inventory +restock

3. 联系供应商
   → 发送飞书消息给采购员

4. 更新库存
   → opc-inventory +update
```

### 场景 3：财务对账

```
1. 查询应收账款
   → opc-finance +receivable

2. 查询应付账款
   → opc-finance +payable

3. 生成财务报表
   → opc-finance +report

4. 导出数据
   → lark-cli base app_table_records export
```

## 模块详情

根据用户需求，读取对应模块的详细文档：

### 客户管理 (opc-customer)

**适用场景**：创建新客户、查询客户信息、更新客户档案、管理客户合同

```bash
# 创建客户
opc-customer +create

# 查询客户
opc-customer +query --company-name "示例公司"

# 创建合同
opc-customer +contract-create
```

详细文档：[`opc-customer/SKILL.md`](opc-customer/SKILL.md)

### 订单管理 (opc-order)

**适用场景**：创建订单、发货处理、开票、回款管理

```bash
# 创建订单
opc-order +create

# 发货
opc-order +ship

# 开票
opc-order +invoice
```

详细文档：[`opc-order/SKILL.md`](opc-order/SKILL.md)

### 库存管理 (opc-inventory)

**适用场景**：查询库存、补货提醒、库存预警

```bash
# 查询库存
opc-inventory +check

# 补货
opc-inventory +restock

# 查看预警
opc-inventory +alert
```

详细文档：[`opc-inventory/SKILL.md`](opc-inventory/SKILL.md)

### 财务管理 (opc-finance)

**适用场景**：应收应付管理、财务报表、成本分析

```bash
# 查询应收
opc-finance +receivable

# 查询应付
opc-finance +payable

# 生成报表
opc-finance +report
```

详细文档：[`opc-finance/SKILL.md`](opc-finance/SKILL.md)

### 人事管理 (opc-hr)

**适用场景**：员工考勤、薪资计算、招聘管理

```bash
# 查询考勤
opc-hr +attendance

# 计算薪资
opc-hr +payroll

# 招聘管理
opc-hr +recruit
```

详细文档：[`opc-hr/SKILL.md`](opc-hr/SKILL.md)

## 权限说明

### 基础权限

所有操作都需要以下飞书开放平台权限：

- `base:app:read` - 读取多维表格数据
- `base:app:write` - 写入多维表格数据

### 特殊权限

- **财务模块**：需要额外审批流程
- **删除操作**：需要管理员权限
- **敏感数据**：仅授权人员可见

## 错误处理

### 常见错误

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| `permission_violations` | 权限不足 | 检查 scope 配置 |
| `table not found` | 数据表不存在 | 先创建数据表 |
| `record not found` | 记录不存在 | 检查 record_id |
| `invalid field` | 字段不存在 | 检查字段名 |

### 处理流程

1. 识别错误类型
2. 读取错误信息中的 `console_url`
3. 在飞书开发者后台配置权限
4. 重新执行操作

## 最佳实践

### 1. 数据一致性

- 所有时间戳使用 Unix 时间戳
- 金额字段统一使用"分"作为单位
- 状态字段使用预定义的枚举值

### 2. 操作审计

- 所有修改操作记录操作人
- 敏感操作记录审批人
- 定期导出操作日志

### 3. 性能优化

- 使用批量操作减少 API 调用
- 合理使用分页参数
- 缓存常用查询结果

## 扩展开发

### 添加新模块

1. 在 `skills/` 目录创建新模块文件夹
2. 创建 `SKILL.md` 文件
3. 创建 `references/` 目录和详细文档
4. 在本文件中添加模块说明

### 自定义工作流

可以组合多个模块创建复杂工作流：

```bash
# 示例：自动化的订单到收款流程
opc-order +create → opc-inventory +check → opc-order +ship
→ opc-order +invoice → opc-finance +payment-receive
```

## 技术支持

- 飞书开放平台文档：https://open.feishu.cn/
- 飞书 CLI 文档：https://github.com/larksuite/cli
- 问题反馈：提交 GitHub Issue

## 许可证

MIT License
