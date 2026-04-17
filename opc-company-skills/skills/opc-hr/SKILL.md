---
name: opc-hr
version: 1.0.0
description: "OPC 人事管理系统：员工考勤、薪资计算、招聘管理。支持员工档案管理、考勤记录查询、薪资自动计算、招聘流程管理。"
metadata:
  category: "erp"
  requires:
    bins: ["lark-cli"]
  cliHelp: "lark-cli base --help"
---

# OPC 人事管理系统

**CRITICAL — 开始前 MUST 先用 Read 工具读取 [`../opc-shared/SKILL.md`](../opc-shared/SKILL.md)，其中包含认证、权限处理**

## 核心概念

### 人事数据模型

```
人事 (HR)
├── 员工档案 (Employees)：基本信息、职位、部门、入职日期
├── 考勤记录 (Attendance)：打卡记录、请假、加班、出差
├── 薪资管理 (Payroll)：基本工资、绩效奖金、扣款、实发工资
└── 招聘管理 (Recruitment)：职位发布、简历筛选、面试安排、录用
```

### 员工生命周期

```
招聘 → 入职 → 在职 → 调岗/晋升 → 离职
```

### 数据表结构

**表名：`opc_employees`**（员工档案）

| 字段名 | 类型 | 说明 | 必填 |
|--------|------|------|------|
| employee_id | text | 员工编号 | ✅ |
| name | text | 姓名 | ✅ |
| department | text | 部门 | ✅ |
| position | text | 职位 | ✅ |
| email | text | 邮箱 | ✅ |
| phone | text | 手机号 | ✅ |
| hire_date | datetime | 入职日期 | ✅ |
| status | select | 员工状态 | ✅ |
| salary | number | 基本工资（分） | ✅ |
| created_at | datetime | 创建时间 | ✅ |

**表名：`opc_attendance`**（考勤记录）

| 字段名 | 类型 | 说明 | 必填 |
|--------|------|------|------|
| attendance_id | text | 考勤编号 | ✅ |
| employee_id | text | 员工编号 | ✅ |
| date | datetime | 日期 | ✅ |
| check_in_time | datetime | 上班打卡时间 | |
| check_out_time | datetime | 下班打卡时间 | |
| work_hours | number | 工作时长（小时） | ✅ |
| status | select | 考勤状态 | ✅ |
| leave_type | select | 请假类型 | |
| overtime_hours | number | 加班时长（小时） | |

## 典型工作流

### 考勤管理流程

```
1. 员工打卡
   → 飞书考勤系统自动记录

2. 同步考勤数据
   → opc-hr +attendance-sync

3. 查询考勤记录
   → opc-hr +attendance-query

4. 计算薪资
   → opc-hr +payroll-calculate
```

### 薪资计算流程

```
1. 获取考勤数据
   → opc-hr +attendance-query

2. 计算基本工资
   → 基本工资 × 出勤天数

3. 计算加班费
   → 加班时长 × 加班费率

4. 计算绩效奖金
   → 绩效评分 × 绩效系数

5. 计算社保公积金
   → 个人部分 + 公司部分

6. 生成工资条
   → opc-hr +payroll-slip
```

### 招聘流程

```
1. 发布职位
   → opc-hr +job-post

2. 收集简历
   → opc-hr +resume-collect

3. 筛选简历
   → opc-hr +resume-screen

4. 安排面试
   → opc-hr +interview-schedule

5. 发送录用通知
   → opc-hr +offer-send

6. 办理入职
   → opc-hr +onboard
```

### CRITICAL — 首次使用任何命令前先查 `-h`

无论是 Shortcut 还是原生 API，**首次调用前必须先运行 `-h` 查看可用参数**：

```bash
# Shortcut
lark-cli base app_table_records search -h

# 原生 API
lark-cli base app_table_records -h
```

## Shortcuts（推荐优先使用）

| Shortcut | 说明 |
|----------|------|
| [`+employee-create`](references/opc-hr-employee-create.md) | 创建员工档案 |
| [`+employee-query`](references/opc-hr-employee-query.md) | 查询员工信息 |
| [`+attendance-sync`](references/opc-hr-attendance-sync.md) | 同步考勤数据 |
| [`+attendance-query`](references/opc-hr-attendance-query.md) | 查询考勤记录 |
| [`+payroll-calculate`](references/opc-hr-payroll-calculate.md) | 计算薪资 |
| [`+payroll-slip`](references/opc-hr-payroll-slip.md) | 生成工资条 |
| [`+job-post`](references/opc-hr-job-post.md) | 发布职位 |
| [`+interview-schedule`](references/opc-hr-interview.md) | 安排面试 |

## 员工状态说明

| 状态 | 说明 |
|------|------|
| **试用期** | 入职3个月内 |
| **正式** | 试用期通过 |
| **离职** | 已离职 |
| **停薪留职** | 暂停工作 |

## 考勤状态说明

| 状态 | 说明 |
|------|------|
| **正常** | 正常出勤 |
| **迟到** | 上班迟到 |
| **早退** | 下班早退 |
| **请假** | 请假（年假/病假/事假） |
| **加班** | 工作日加班 |
| **出差** | 出差 |
| **缺勤** | 未打卡且未请假 |

## 薪资计算规则

### 基本工资计算

```
基本工资 = 月基本工资 × (出勤天数 / 当月工作日)
```

### 加班费计算

```
工作日加班 = 加班时长 × 时薪 × 1.5
周末加班 = 加班时长 × 时薪 × 2
节假日加班 = 加班时长 × 时薪 × 3

时薪 = 月基本工资 / (21.75 × 8)
```

### 绩效奖金计算

```
绩效奖金 = 基本工资 × 绩效系数 × (出勤天数 / 当月工作日)

绩效系数：
- S级：1.2
- A级：1.0
- B级：0.8
- C级：0.6
```

### 社保公积金

```
个人部分 = 基本工资 × 个人缴费比例
公司部分 = 基本工资 × 公司缴费比例

具体比例按当地政策执行
```

## API Resources

### 查询员工信息

```bash
lark-cli base app_table_records search \
  --table-id opc_employees \
  --as bot \
  --data '{
    "filter": {
      "conditions": [{
        "field_name": "employee_id",
        "operator": "is",
        "value": ["EMP-001"]
      }]
    }
  }'
```

### 查询考勤记录

```bash
lark-cli base app_table_records search \
  --table-id opc_attendance \
  --as bot \
  --data '{
    "filter": {
      "logic": "and",
      "conditions": [
        {
          "field_name": "employee_id",
          "operator": "is",
          "value": ["EMP-001"]
        },
        {
          "field_name": "date",
          "operator": "greaterThanOrEqual",
          "value": [1741564800]
        }
      ]
    }
  }'
```

### 更新员工信息

```bash
lark-cli base app_table_records update \
  --table-id opc_employees \
  --record-id <record_id> \
  --as bot \
  --data '{
    "fields": {
      "position": "高级工程师",
      "salary": 300000,
      "updated_at": 1741564800
    }
  }'
```

## 权限说明

### 数据权限

| 角色 | 可查看数据 | 可操作 |
|------|-----------|--------|
| **员工** | 仅自己的信息 | 查看考勤、工资条 |
| **主管** | 本部门员工信息 | 查看考勤、审批请假 |
| **HR** | 所有员工信息 | 全部操作 |
| **财务** | 薪资相关数据 | 查看薪资、发放工资 |

### 操作权限

| 操作 | 所需 scope | 审批要求 |
|------|-----------|---------|
| 查询员工 | `base:app:read` | - |
| 创建员工 | `base:app:write` | HR审批 |
| 更新员工 | `base:app:write` | HR审批 |
| 查询考勤 | `base:app:read` | - |
| 计算薪资 | `base:app:read` | - |
| 发放工资 | `base:app:write` | 财务审批 |

## 常见场景

### 场景 1：新员工入职

```
1. 创建员工档案
   → opc-hr +employee-create

2. 分配工位和设备
   → 行政系统

3. 开通飞书账号
   → 飞书管理后台

4. 添加到组织架构
   → 飞书通讯录
```

### 场景 2：月度考勤统计

```
1. 同步考勤数据
   → opc-hr +attendance-sync

2. 查询异常考勤
   → opc-hr +attendance-query --status abnormal

3. 确认考勤记录
   → 通知员工确认

4. 计算薪资
   → opc-hr +payroll-calculate
```

### 场景 3：招聘流程

```
1. 发布职位
   → opc-hr +job-post

2. 收集简历
   → 招聘网站/邮箱

3. 筛选简历
   → opc-hr +resume-screen

4. 安排面试
   → opc-hr +interview-schedule

5. 发送录用通知
   → opc-hr +offer-send

6. 办理入职
   → opc-hr +onboard
```

## 相关 Skill

- 财务管理：`../opc-finance/SKILL.md`
- 行政管理（待开发）：`../opc-admin/SKILL.md`
