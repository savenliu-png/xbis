# 页面模板（layout/）

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 页面模板（layout/） |
| 任务编号 | T004 |
| 所属模块 ⭐ | M1 基础设计系统 |
| 优先级 | P0 |
| 指派给 | @frontend-architect |
| 预计工期 | 2 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-05 |

## 2. 需求背景

### 2.1 问题描述
当前项目页面结构不统一，各页面自行处理布局、加载态、空态、错误态，导致：
- 页面结构不一致，用户体验差
- 重复实现相同布局逻辑
- 状态管理散落在各页面

### 2.2 目标用户
- 前端开发者：使用页面模板快速搭建页面
- 产品经理：确保页面结构一致性

### 2.3 预期效果
- 建立 5 个页面模板，覆盖常用页面类型
- 所有模板内置状态机（loading/empty/error/idle）
- 提供统一的页面布局和样式

## 3. 页面位置 ⭐

### 3.1 用户端
| 页面 | 路由 | 入口 |
|------|------|------|
| 全局模板 | — | 所有页面 |

### 3.2 管理端
| 页面 | 路由 | 入口 |
|------|------|------|
| 全局模板 | — | 所有页面 |

### 3.3 页面原型/设计稿
- 参考：`packages/components/layout/index.ts`

## 4. 功能范围

### 4.1 包含功能
- [ ] ListPageShell（列表页模板）
- [ ] DetailPageShell（详情页模板）
- [ ] FormPageShell（表单页模板）
- [ ] TaskPageShell（任务页模板）
- [ ] AbilityPageShell（能力页模板）

### 4.2 不包含功能（明确排除）
- 具体业务逻辑（由页面实现）
- 数据获取逻辑（由页面实现）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 页面状态
export type PageState = 'idle' | 'loading' | 'empty' | 'error';

// 列表页模板 Props
export interface ListPageShellProps {
  title: string;
  subtitle?: string;
  actions?: React.ReactNode;
  filters?: React.ReactNode;
  state: PageState;
  error?: Error;
  onRetry?: () => void;
  children: React.ReactNode;
}

// 详情页模板 Props
export interface DetailPageShellProps {
  title: string;
  subtitle?: string;
  backLink?: string;
  actions?: React.ReactNode;
  tabs?: { key: string; label: string; content: React.ReactNode }[];
  state: PageState;
  error?: Error;
  onRetry?: () => void;
  children: React.ReactNode;
}

// 表单页模板 Props
export interface FormPageShellProps {
  title: string;
  subtitle?: string;
  backLink?: string;
  actions?: React.ReactNode;
  state: PageState;
  error?: Error;
  onRetry?: () => void;
  children: React.ReactNode;
}

// 任务页模板 Props
export interface TaskPageShellProps {
  title: string;
  subtitle?: string;
  actions?: React.ReactNode;
  filters?: React.ReactNode;
  stats?: React.ReactNode;
  state: PageState;
  error?: Error;
  onRetry?: () => void;
  children: React.ReactNode;
}

// 能力页模板 Props
export interface AbilityPageShellProps {
  title: string;
  subtitle?: string;
  categories?: React.ReactNode;
  filters?: React.ReactNode;
  state: PageState;
  error?: Error;
  onRetry?: () => void;
  children: React.ReactNode;
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/components/layout/*/types.ts`
- 命名：`ListPageShellProps`, `DetailPageShellProps`, `FormPageShellProps`, `TaskPageShellProps`, `AbilityPageShellProps`

## 6. 交互流程

### 6.1 主流程
```
[页面使用模板] ──► [传入状态和子元素] ──► [模板渲染布局] ──► [状态变化时更新展示]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 加载失败 | 数据请求失败 | 展示错误状态 | 展示错误信息和重试按钮 |
| 数据为空 | 返回空数组 | 展示空状态 | 展示空状态插图和提示 |

### 6.3 边界情况
- 无标题：展示默认标题
- 无操作：不展示操作区
- 无筛选：不展示筛选区

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Spinner | 加载状态 | 否（T002） |
| Empty | 空状态 | 否（T002） |
| Alert | 错误状态 | 否（T002） |
| Button | 重试按钮 | 否（T002） |
| Tabs | 详情页标签 | 否（T002） |

### 7.2 业务组件（business/）
无

### 7.3 页面块组件（blocks/）
无

### 7.4 页面模板（layout/）
无（自身就是模板层）

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| ListPageShell | layout | 列表页模板 |
| DetailPageShell | layout | 详情页模板 |
| FormPageShell | layout | 表单页模板 |
| TaskPageShell | layout | 任务页模板 |
| AbilityPageShell | layout | 能力页模板 |

## 8. API 接口 ⭐

无

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] ListPageShell 支持标题/副标题/操作区/筛选区/状态切换
- [ ] DetailPageShell 支持标题/副标题/返回链接/操作区/标签页/状态切换
- [ ] FormPageShell 支持标题/副标题/返回链接/操作区/状态切换
- [ ] TaskPageShell 支持标题/副标题/操作区/筛选区/统计区/状态切换
- [ ] AbilityPageShell 支持标题/副标题/分类区/筛选区/状态切换
- [ ] 所有模板支持 loading/empty/error/idle 四态
- [ ] 错误状态展示错误信息和重试按钮
- [ ] 空状态展示空状态插图和提示

### 9.2 技术验收
- [ ] TypeScript 类型完整，无 `any`
- [ ] 使用 Design Tokens，无散落样式
- [ ] 模板 props 有 JSDoc 注释
- [ ] 支持 ref 转发

### 9.3 性能验收
- [ ] 模板渲染 < 50ms
- [ ] 状态切换无闪烁

### 9.4 兼容性验收
- [ ] 支持 Chrome/Firefox/Safari/Edge 最新 2 个版本

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 全新功能，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T001` — Design Tokens 系统 — 待开发
  - [x] `T002` — 基础组件库 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 模板过于僵化 | 中 | 中 | 提供灵活的 slots 机制 |
| 状态机复杂 | 低 | 低 | 使用简单的状态枚举 |

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

- 模板只负责布局和状态展示，不处理业务逻辑
- 子元素通过 children 传入，保持灵活性
- 状态机使用简单的 useState 实现

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
