# T002 基础组件库 — 修正版设计方案（V2）

> 原设计方案: [T002-base-components-spec.md](../T002-base-components-spec.md)
> 评审报告: [T002-base-components-Reviewer.md](../T002-base-components-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed with changes

---

## 二、必须修改项（Blocking）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 明确二次封装的价值点 | 第 2 节组件拆分 |
| 2 | 补充缺失的基础表单组件（Select/DatePicker/Switch） | 第 2 节组件拆分 |
| 3 | 确定并统一样式方案为 CSS Modules | 第 2 节组件拆分 |

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 将 CopyButton 移至 business 层 | 第 2 节组件拆分 |
| 2 | 明确 StatusDot 与 Badge 的边界 | 第 2 节组件拆分 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

无页面结构，本任务为组件库建设。

---

### 2. 组件拆分

#### base/ 层组件（V2 完整列表）【已修复】

| 组件 | 来源 | 新增 Props | 封装价值点 |
|------|------|-----------|-----------|
| Button | Ant Design | `variant`, `isLoading` | 统一品牌色、加载状态样式 |
| Input | Ant Design | `size`, `status` | 统一边框圆角、错误态颜色 |
| Textarea | Ant Design | `maxLength`, `showCount` | 统一字符计数样式 |
| Select | Ant Design 【新增】 | `size`, `status` | 统一选择器样式 |
| DatePicker | Ant Design 【新增】 | `format`, `showTime` | 统一日期格式、暗色适配 |
| Switch | Ant Design 【新增】 | `size`, `checkedChildren` | 统一开关尺寸、品牌色 |
| Checkbox | Ant Design 【新增】 | - | 统一复选框样式 |
| Radio | Ant Design 【新增】 | - | 统一单选框样式 |
| Upload | Ant Design 【新增】 | `accept`, `maxSize` | 统一上传样式、文件校验 |
| Table | Ant Design | `striped`, `hoverable` | 统一表格行样式 |
| Tag | Ant Design | `variant`, `colorScheme` | 统一标签色系 |
| Badge | Ant Design | `dot`, `status` | 统一徽标样式 |
| Modal | Ant Design | `size`, `hideClose` | 统一弹窗尺寸、关闭按钮 |
| Drawer | Ant Design | `size`, `placement` | 统一抽屉尺寸 |
| Tooltip | Ant Design | `variant` | 统一提示样式 |
| Popover | Ant Design | - | 统一气泡样式 |
| Tabs | Ant Design | `variant`, `size` | 统一标签页样式 |
| Steps | Ant Design | `direction`, `size` | 统一步骤条样式 |
| Pagination | Ant Design | `simple`, `showSizeChanger` | 统一分页样式 |
| Form | Ant Design | `layout`, `size` | 统一表单布局 |
| Skeleton | Ant Design | `variant` | 统一骨架屏样式 |
| Empty | Ant Design | `image`, `description` | 统一空态样式 |
| StatusDot | 自研 | `status`, `pulse` | 状态指示（与 Badge 区分：Badge 用于徽标计数，StatusDot 用于状态指示）【已修复】 |

#### 分层调整【已修复】

- **CopyButton** → 移至 `business/` 层（含剪贴板操作和反馈状态的业务逻辑）
- **StatusDot** → 明确边界：仅用于状态指示（在线/离线/处理中），支持 pulse 动画；Badge 用于徽标、计数、红点通知

#### 样式方案【已修复】

统一使用 **CSS Modules** 方案：
- 与 Ant Design 的 CSS-in-JS 解耦
- 更好的 Tree-shaking 支持
- 更易于主题切换
- 文件命名：`ComponentName.module.css`

---

### 3. 数据流

无运行时数据流。

---

### 4. 状态管理

无状态管理需求。

---

### 5. API 调用

无 API 调用。

---

### 6. 用户交互流程

无用户交互流程。

---

### 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| Ant Design 版本升级 | 封装层隔离变更，仅修改 base 层 |
| Token 未定义 | 使用 Ant Design 默认 fallback |

---

### 8. 性能优化

- 按需导出，支持 Tree-shaking
- CSS Modules 支持浏览器缓存

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 封装过度 | 中 | 过度封装导致灵活性降低 | 保留 Ant Design 原始 Props 透传 |
| 样式冲突 | 低 | CSS Modules 与 Ant Design 样式冲突 | 使用 `:global` 谨慎覆盖 |

---

### 10. 开发步骤拆分

#### Step 1: 核心表单组件（1 人日）【已修复】
- [ ] Button / Input / Textarea
- [ ] Select / DatePicker / Switch / Checkbox / Radio / Upload 【新增】

#### Step 2: 展示组件（1 人日）
- [ ] Table / Tag / Badge / StatusDot
- [ ] Skeleton / Empty

#### Step 3: 反馈组件（0.5 人日）
- [ ] Modal / Drawer / Tooltip / Popover

#### Step 4: 导航组件（0.5 人日）
- [ ] Tabs / Steps / Pagination

#### Step 5: 表单容器（0.5 人日）
- [ ] Form

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **基础表单组件** | 20 个（无 Select/DatePicker/Switch） | 24 个（新增 Select/DatePicker/Switch/Checkbox/Radio/Upload） |
| **CopyButton** | 在 base 层 | 移至 business 层 |
| **StatusDot/Badge 边界** | 未明确 | 明确区分：StatusDot 用于状态指示，Badge 用于徽标计数 |
| **样式方案** | 未明确 | 统一使用 CSS Modules |
| **封装价值点** | 未明确 | 每个组件明确说明新增 Props 和封装价值 |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（补充基础表单组件、明确样式方案、明确封装价值点）
2. 组件分层已调整（CopyButton 移至 business 层）
3. StatusDot 与 Badge 边界已明确
4. 风险等级：中（可控）
