# T002 基础组件库 — 最终可开发设计方案

> 任务卡: [T002-base-components.md](../T002-base-components.md)
> 设计输入: docs/component-architecture.md, docs/dev-rules.md, docs/design-tokens.md
> 输出日期: 2026-04-24
> 文档版本: Final

---

## 1. 页面结构

本任务为**基础能力层**建设，无页面结构。输出物为组件库代码包。

```
packages/components/base/           # 基础组件包
├── index.ts                        # 统一导出入口
├── Button/
│   ├── index.tsx
│   └── style.module.css
├── Input/
│   ├── index.tsx
│   └── style.module.css
├── Textarea/
│   ├── index.tsx
│   └── style.module.css
├── Select/
│   ├── index.tsx
│   └── style.module.css
├── DatePicker/
│   ├── index.tsx
│   └── style.module.css
├── Switch/
│   ├── index.tsx
│   └── style.module.css
├── Checkbox/
│   ├── index.tsx
│   └── style.module.css
├── Radio/
│   ├── index.tsx
│   └── style.module.css
├── Upload/
│   ├── index.tsx
│   └── style.module.css
├── Card/
│   ├── index.tsx
│   └── style.module.css
├── Tag/
│   ├── index.tsx
│   └── style.module.css
├── Badge/
│   ├── index.tsx
│   └── style.module.css
├── Empty/
│   ├── index.tsx
│   └── style.module.css
├── Skeleton/
│   ├── index.tsx
│   └── style.module.css
├── Avatar/
│   ├── index.tsx
│   └── style.module.css
├── Tooltip/
│   ├── index.tsx
│   └── style.module.css
├── Modal/
│   ├── index.tsx
│   └── style.module.css
├── Drawer/
│   ├── index.tsx
│   └── style.module.css
├── Table/
│   ├── index.tsx
│   └── style.module.css
├── Tabs/
│   ├── index.tsx
│   └── style.module.css
├── Steps/
│   ├── index.tsx
│   └── style.module.css
├── Pagination/
│   ├── index.tsx
│   └── style.module.css
├── Form/
│   ├── index.tsx
│   └── style.module.css
├── Alert/
│   ├── index.tsx
│   └── style.module.css
├── StatusDot/
│   ├── index.tsx
│   └── style.module.css
└── Divider/
    ├── index.tsx
    └── style.module.css
```

---

## 2. 组件拆分

### base/ 层组件（24 个）

| 组件 | 来源 | 新增 Props | 封装价值点 |
|------|------|-----------|-----------|
| Button | Ant Design | `variant`, `isLoading` | 统一品牌色、加载状态样式 |
| Input | Ant Design | `size`, `status` | 统一边框圆角、错误态颜色 |
| Textarea | Ant Design | `maxLength`, `showCount` | 统一字符计数样式 |
| Select | Ant Design | `size`, `status` | 统一选择器样式 |
| DatePicker | Ant Design | `format`, `showTime` | 统一日期格式、暗色适配 |
| Switch | Ant Design | `size`, `checkedChildren` | 统一开关尺寸、品牌色 |
| Checkbox | Ant Design | - | 统一复选框样式 |
| Radio | Ant Design | - | 统一单选框样式 |
| Upload | Ant Design | `accept`, `maxSize` | 统一上传样式、文件校验 |
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
| StatusDot | 自研 | `status`, `pulse` | 状态指示（与 Badge 区分：Badge 用于徽标计数，StatusDot 用于状态指示） |
| Divider | Ant Design | `variant` | 统一分割线样式 |

### 样式方案

统一使用 **CSS Modules** 方案：
- 与 Ant Design 的 CSS-in-JS 解耦
- 更好的 Tree-shaking 支持
- 更易于主题切换
- 文件命名：`ComponentName.module.css`

---

## 3. 数据流

无业务数据流。组件为纯展示型，数据通过 Props 传入。

```
父组件 ──► Props ──► Base Component ──► UI 渲染
```

---

## 4. 状态管理

- 组件内部状态：使用 `useState` 管理交互状态
- 无全局状态依赖

---

## 5. API 调用

无 API 调用。

---

## 6. 用户交互流程

| 组件 | 交互 | 行为 | 结果 |
|------|------|------|------|
| Button | 点击 | onClick 回调 | 执行业务逻辑 |
| Input | 输入 | onChange 回调 | 更新表单值 |
| Modal | 点击遮罩/关闭按钮 | onCancel 回调 | 关闭弹窗 |
| Search | 输入/回车 | onSearch 回调 | 触发搜索 |

---

## 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| 缺少必填 Props | TypeScript 编译时检查 |
| 子组件为空 | 渲染 Empty 组件 |
| 样式 Token 缺失 | 使用 Ant Design 默认值 fallback |
| Ant Design 版本升级 | 封装层隔离变更，仅修改 base 层 |

---

## 8. 性能优化

- 组件使用 `React.memo` 避免不必要的重渲染
- 样式使用 CSS Modules，避免全局污染
- 支持 Tree-shaking（按需导出）
- CSS Modules 支持浏览器缓存

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 与 Ant Design 样式冲突 | 中 | 二次封装可能覆盖 Ant Design 默认样式 | 使用 `ConfigProvider` 透传 Token |
| 组件 API 不兼容 | 中 | 新增 Props 可能与 Ant Design 未来版本冲突 | 使用 `extends` 继承原 Props |
| 暗色模式适配遗漏 | 中 | 新增组件可能遗漏暗色模式样式 | 建立组件审查清单 |
| 封装过度 | 中 | 过度封装导致灵活性降低 | 保留 Ant Design 原始 Props 透传 |
| 样式冲突 | 低 | CSS Modules 与 Ant Design 样式冲突 | 使用 `:global` 谨慎覆盖 |

---

## 10. 开发步骤拆分

### Step 1: 核心表单组件（1 人日）
- [ ] Button / Input / Textarea
- [ ] Select / DatePicker / Switch / Checkbox / Radio / Upload

### Step 2: 展示组件（1 人日）
- [ ] Table / Tag / Badge / StatusDot
- [ ] Skeleton / Empty

### Step 3: 反馈组件（0.5 人日）
- [ ] Modal / Drawer / Tooltip / Popover

### Step 4: 导航组件（0.5 人日）
- [ ] Tabs / Steps / Pagination

### Step 5: 表单容器（0.5 人日）
- [ ] Form

### Step 6: 统一导出 & 文档（1 人日）
- [ ] 创建 `index.ts` 统一导出
- [ ] 编写组件使用文档
- [ ] 编写 Storybook stories（可选）
