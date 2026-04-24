# 管理端概览页

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 管理端概览页 |
| 任务编号 | T025 |
| 所属模块 ⭐ | M9 概览 |
| 优先级 | P1 |
| 指派给 | @frontend-dev |
| 预计工期 | 3 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-15 |

## 2. 需求背景

### 2.1 问题描述
当前项目缺乏管理端概览页面，管理员需要：
- 查看任务量统计
- 查看收入统计
- 查看成功率
- 查看执行器健康

### 2.2 目标用户
- 管理员：监控平台运行状态
- 运营人员：查看业务数据

### 2.3 预期效果
- 提供清晰的数据概览
- 支持图表展示
- 支持实时数据

## 3. 页面位置 ⭐

### 3.1 用户端
无

### 3.2 管理端
| 页面 | 路由 | 入口 |
|------|------|------|
| 管理概览 | `/admin/overview` | 管理端首页 |

### 3.3 页面原型/设计稿
- 参考：`packages/pages/admin/AdminOverview`

## 4. 功能范围

### 4.1 包含功能
- [ ] 任务量统计卡片
- [ ] 收入统计卡片
- [ ] 成功率统计卡片
- [ ] 执行器健康状态
- [ ] 趋势图表
- [ ] 最近任务列表
- [ ] 告警信息
- [ ] 空态/加载态/错误态

### 4.2 不包含功能（明确排除）
- 详细报表（管理端功能）
- 数据导出（V2 迭代）
- 自定义仪表盘（V2 迭代）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 概览统计
export interface OverviewStats {
  totalJobs: number;
  jobsToday: number;
  jobsGrowth: number;
  totalRevenue: number;
  revenueToday: number;
  revenueGrowth: number;
  successRate: number;
  successRateChange: number;
  activeExecutors: number;
  degradedExecutors: number;
  offlineExecutors: number;
}

// 趋势数据
export interface TrendData {
  date: string;
  jobs: number;
  revenue: number;
  successRate: number;
}

// 最近任务
export interface RecentJob {
  id: string;
  jobId: string;
  abilityName: string;
  status: JobStatus;
  createdAt: string;
}

// 告警信息
export interface AlertInfo {
  id: string;
  level: 'info' | 'warning' | 'critical';
  message: string;
  source: string;
  createdAt: string;
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`OverviewStats`, `TrendData`, `RecentJob`, `AlertInfo`

## 6. 交互流程

### 6.1 主流程
```
[管理员进入概览页] ──► [加载统计数据] ──► [展示图表] ──► [加载最近任务] ──► [展示告警]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 加载失败 | 网络错误 | 显示错误态 | 提示刷新 |
| 无数据 | 新平台无数据 | 显示空态 | 提示正常 |
| 权限不足 | 非管理员 | 跳转 403 | 提示无权限 |

### 6.3 边界情况
- 大量数据：图表聚合
- 实时更新：定时刷新
- 暗色模式：自动适配

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Card | 统计卡片 | 是（T002） |
| Badge | 状态标识 | 是（T002） |
| Table | 任务表格 | 是（T002） |
| Skeleton | 加载骨架 | 是（T002） |
| Empty | 空态展示 | 是（T002） |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| StatCard | 统计卡片 | 否 |
| TrendChart | 趋势图表 | 否 |

### 7.3 页面块组件（blocks/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| StatsGrid | 统计网格 | 否 |
| TrendChart | 趋势图区 | 否 |
| RecentJobs | 最近任务区 | 否 |
| AlertList | 告警列表区 | 否 |

### 7.4 页面模板（layout/）
| 模板 | 用途 |
|------|------|
| AdminPageShell | 管理页骨架 |

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| StatCard | business | 统计卡片 |
| TrendChart | business | 趋势图表 |
| StatsGrid | blocks | 统计网格 |
| RecentJobs | blocks | 最近任务区 |
| AlertList | blocks | 告警列表区 |

## 8. API 接口 ⭐

### 8.1 新增接口

#### 概览统计查询
- **Method**: GET
- **Path**: `/admin-api/v1/overview/stats`
- **响应类型**: `OverviewStats`
- **权限**: 管理员

**响应示例**:
```json
{
  "success": true,
  "data": {
    "totalJobs": 125800,
    "jobsToday": 1250,
    "jobsGrowth": 12.5,
    "totalRevenue": 56800,
    "revenueToday": 520,
    "revenueGrowth": 8.3,
    "successRate": 98.5,
    "successRateChange": 0.5,
    "activeExecutors": 8,
    "degradedExecutors": 1,
    "offlineExecutors": 1
  }
}
```

#### 趋势数据查询
- **Method**: GET
- **Path**: `/admin-api/v1/overview/trends`
- **请求类型**: `{ period: 'daily' | 'weekly' | 'monthly' }`
- **响应类型**: `TrendData[]`
- **权限**: 管理员

#### 最近任务查询
- **Method**: GET
- **Path**: `/admin-api/v1/overview/recent-jobs`
- **响应类型**: `{ items: RecentJob[]; total: number }`
- **权限**: 管理员

#### 告警信息查询
- **Method**: GET
- **Path**: `/admin-api/v1/overview/alerts`
- **响应类型**: `{ items: AlertInfo[]; total: number }`
- **权限**: 管理员

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`adminApi.overview.stats`, `adminApi.overview.trends`, `adminApi.overview.recentJobs`, `adminApi.overview.alerts`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 任务量统计展示正常
- [ ] 收入统计展示正常
- [ ] 成功率统计展示正常
- [ ] 执行器健康状态展示正常
- [ ] 趋势图表展示正常
- [ ] 最近任务列表展示正常
- [ ] 告警信息展示正常
- [ ] 空态/加载态/错误态完整

### 9.2 技术验收
- [ ] TypeScript 类型完整，无 `any`
- [ ] 使用 Design Tokens，无散落样式
- [ ] 使用页面模板骨架
- [ ] 页面状态机完整（loading/empty/error/idle）
- [ ] 组件复用符合分层规范
- [ ] API 错误处理完整

### 9.3 性能验收
- [ ] 首屏加载 < 2s
- [ ] 图表渲染 < 500ms
- [ ] 无内存泄漏（useEffect 清理）

### 9.4 兼容性验收
- [ ] 桌面端 ≥1200px 正常显示
- [ ] 管理端无需移动端设计

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 全新功能，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T003` — 业务组件库 — 待开发
  - [x] `T004` — 页面模板 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 数据量大 | 中 | 中 | 数据聚合 |
| 实时更新 | 中 | 低 | 定时刷新 |
| 图表性能 | 中 | 中 | 限制数据点 |

## 11. 开发检查清单

### 11.1 开发前
- [ ] 需求已评审并确认
- [ ] 设计稿已确认
- [ ] 数据结构和 API 已评审
- [ ] 依赖任务已完成

### 11.2 开发中
- [ ] 遵循 `docs/dev-rules.md` 规范
- [ ] 遵循 `AI_RULES.md` 规范
- [ ] 自测覆盖所有验收标准

### 11.3 开发后
- [ ] 代码已提交 PR
- [ ] PR 已通过 Code Review
- [ ] 已联调通过
- [ ] 已更新文档（如需要）

## 12. 备注

- 管理端无需移动端设计
- 数据定时刷新（5 分钟）
- 支持暗色模式

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
