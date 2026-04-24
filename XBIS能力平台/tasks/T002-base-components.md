# 基础组件库（base/）

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 基础组件库（base/） |
| 任务编号 | T002 |
| 所属模块 ⭐ | M1 基础设计系统 |
| 优先级 | P0 |
| 指派给 | @frontend-architect |
| 预计工期 | 5 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-02 |

## 2. 需求背景

### 2.1 问题描述
当前项目使用 Ant Design 5.x 组件，但缺乏统一的二次封装，导致：
- 各页面组件使用方式不一致
- 样式覆盖散落在各处
- 暗色模式适配困难

### 2.2 目标用户
- 前端开发者：使用统一的基础组件构建页面
- UI 设计师：确保组件视觉一致性

### 2.3 预期效果
- 建立 20+ 基础组件，覆盖常用 UI 元素
- 所有组件基于 Design Tokens，支持暗色模式
- 提供 Storybook 文档，方便查阅

## 3. 页面位置 ⭐

### 3.1 用户端
| 页面 | 路由 | 入口 |
|------|------|------|
| 全局组件 | — | 所有页面 |

### 3.2 管理端
| 页面 | 路由 | 入口 |
|------|------|------|
| 全局组件 | — | 所有页面 |

### 3.3 页面原型/设计稿
- 参考：`packages/components/base/index.ts`
- Storybook: `http://localhost:6006`

## 4. 功能范围

### 4.1 包含功能
- [ ] Button（按钮）
- [ ] Card（卡片）
- [ ] Input（输入框）
- [ ] TextArea（文本域）
- [ ] Select（选择器）
- [ ] Checkbox（复选框）
- [ ] Radio（单选框）
- [ ] Switch（开关）
- [ ] Tag（标签）
- [ ] Badge（徽标）
- [ ] Modal（弹窗）
- [ ] Drawer（抽屉）
- [ ] Table（表格）
- [ ] Pagination（分页）
- [ ] Tabs（标签页）
- [ ] Alert（警告提示）
- [ ] Empty（空状态）
- [ ] Skeleton（骨架屏）
- [ ] Spinner（加载中）
- [ ] Tooltip（文字提示）
- [ ] Popover（气泡卡片）
- [ ] Dropdown（下拉菜单）

### 4.2 不包含功能（明确排除）
- 业务逻辑组件（由 business/ 层实现）
- 页面级组件（由 blocks/ 层实现）
- 复杂表单组件（如 FormBuilder，由 blocks/ 层实现）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// Button Props
export interface ButtonProps {
  variant?: 'solid' | 'outline' | 'ghost' | 'link';
  size?: 'sm' | 'md' | 'lg';
  color?: 'brand' | 'success' | 'warning' | 'error' | 'neutral';
  loading?: boolean;
  disabled?: boolean;
  icon?: React.ReactNode;
  children: React.ReactNode;
  onClick?: () => void;
}

// Card Props
export interface CardProps {
  title?: string;
  subtitle?: string;
  extra?: React.ReactNode;
  footer?: React.ReactNode;
  padding?: 'none' | 'sm' | 'md' | 'lg';
  shadow?: 'none' | 'sm' | 'md' | 'lg';
  children: React.ReactNode;
}

// Input Props
export interface InputProps {
  type?: 'text' | 'password' | 'email' | 'number';
  size?: 'sm' | 'md' | 'lg';
  status?: 'default' | 'error' | 'warning' | 'success';
  prefix?: React.ReactNode;
  suffix?: React.ReactNode;
  placeholder?: string;
  value?: string;
  onChange?: (value: string) => void;
}

// Tag Props
export interface TagProps {
  color?: 'brand' | 'success' | 'warning' | 'error' | 'neutral' | 'info';
  size?: 'sm' | 'md';
  variant?: 'solid' | 'outline' | 'light';
  closable?: boolean;
  children: React.ReactNode;
  onClose?: () => void;
}

// Modal Props
export interface ModalProps {
  open: boolean;
  title?: string;
  width?: number | string;
  closable?: boolean;
  maskClosable?: boolean;
  footer?: React.ReactNode | null;
  onCancel?: () => void;
  onOk?: () => void;
  children: React.ReactNode;
}

// Table Props
export interface TableProps<T> {
  dataSource: T[];
  columns: Column<T>[];
  loading?: boolean;
  pagination?: PaginationConfig | false;
  rowKey?: string | ((record: T) => string);
  onRowClick?: (record: T) => void;
  emptyText?: string;
}

// Empty Props
export interface EmptyProps {
  image?: React.ReactNode;
  title?: string;
  description?: string;
  action?: React.ReactNode;
}

// Alert Props
export interface AlertProps {
  type?: 'info' | 'success' | 'warning' | 'error';
  title?: string;
  message: string;
  closable?: boolean;
  onClose?: () => void;
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/components/base/*/types.ts`
- 命名：`ButtonProps`, `CardProps`, `InputProps`, `TagProps`, `ModalProps`, `TableProps`, `EmptyProps`, `AlertProps`

## 6. 交互流程

### 6.1 主流程
```
[开发者引入组件] ──► [传入 props] ──► [组件渲染] ──► [用户交互] ──► [触发回调]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 必填 props 缺失 | 开发时未传入必要 props | TypeScript 编译报错 | 开发时即发现 |
| 非法 props 值 | 传入不在枚举范围内的值 | TypeScript 编译报错 | 开发时即发现 |

### 6.3 边界情况
- 无子元素：组件展示占位符或空状态
- 超长内容：支持 ellipsis 或滚动
- 暗色模式：自动适配暗色主题

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
无（自身就是基础组件层）

### 7.2 业务组件（business/）
无

### 7.3 页面块组件（blocks/）
无

### 7.4 页面模板（layout/）
无

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| Button | base | 按钮 |
| Card | base | 卡片 |
| Input | base | 输入框 |
| TextArea | base | 文本域 |
| Select | base | 选择器 |
| Checkbox | base | 复选框 |
| Radio | base | 单选框 |
| Switch | base | 开关 |
| Tag | base | 标签 |
| Badge | base | 徽标 |
| Modal | base | 弹窗 |
| Drawer | base | 抽屉 |
| Table | base | 表格 |
| Pagination | base | 分页 |
| Tabs | base | 标签页 |
| Alert | base | 警告提示 |
| Empty | base | 空状态 |
| Skeleton | base | 骨架屏 |
| Spinner | base | 加载中 |
| Tooltip | base | 文字提示 |
| Popover | base | 气泡卡片 |
| Dropdown | base | 下拉菜单 |

## 8. API 接口 ⭐

无

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] Button 支持 4 种 variant（solid/outline/ghost/link）和 4 种 size
- [ ] Card 支持 title/subtitle/extra/footer，可配置 padding 和 shadow
- [ ] Input 支持 prefix/suffix，错误状态样式
- [ ] Tag 支持 6 种 color 和 3 种 variant
- [ ] Modal 支持自定义 width/footer，maskClosable
- [ ] Table 支持排序/筛选/分页，loading 状态
- [ ] Empty 支持自定义图片和 action
- [ ] Alert 支持 4 种 type，可关闭
- [ ] 所有组件支持暗色模式
- [ ] 所有组件有 Storybook 示例

### 9.2 技术验收
- [ ] TypeScript 类型完整，无 `any`
- [ ] 使用 Design Tokens，无散落样式
- [ ] 组件 props 有 JSDoc 注释
- [ ] 组件导出符合命名规范
- [ ] 支持 ref 转发

### 9.3 性能验收
- [ ] 组件首屏渲染 < 50ms
- [ ] 无不必要的重渲染

### 9.4 兼容性验收
- [ ] 支持 Chrome/Firefox/Safari/Edge 最新 2 个版本

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 全新功能，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T001` — Design Tokens 系统 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| Ant Design 样式覆盖冲突 | 中 | 中 | 使用 CSS-in-JS 或 CSS 变量隔离 |
| 暗色模式适配遗漏 | 中 | 高 | 建立暗色模式检查清单 |

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

- 基于 Ant Design 5.x 二次封装，保持 API 兼容
- 优先实现高频组件（Button/Card/Input/Tag/Modal/Table/Empty/Alert）
- 低频组件可后续迭代补充

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
