# T002 基础组件库 — 工程级设计方案

> 任务卡: [T002-base-components.md](../T002-base-components.md)  
> 设计输入: docs/component-architecture.md, docs/dev-rules.md, docs/design-tokens.md  
> 输出日期: 2026-04-24

---

## 1. 页面结构

本任务为**基础能力层**建设，无页面结构。输出物为组件库代码包。

```
packages/components/base/           # 基础组件包
├── index.ts                        # 统一导出入口
├── Button/
│   ├── index.tsx
│   └── style.ts
├── Card/
│   ├── index.tsx
│   └── style.ts
├── Input/
│   ├── index.tsx
│   └── style.ts
├── Tag/
│   ├── index.tsx
│   └── style.ts
├── Badge/
│   ├── index.tsx
│   └── style.ts
├── Empty/
│   ├── index.tsx
│   └── style.ts
├── Spinner/
│   ├── index.tsx
│   └── style.ts
├── Avatar/
│   ├── index.tsx
│   └── style.ts
├── Tooltip/
│   ├── index.tsx
│   └── style.ts
├── Modal/
│   ├── index.tsx
│   └── style.ts
├── Drawer/
│   ├── index.tsx
│   └── style.ts
├── Table/
│   ├── index.tsx
│   └── style.ts
├── Tabs/
│   ├── index.tsx
│   └── style.ts
├── Alert/
│   ├── index.tsx
│   └── style.ts
├── Form/
│   ├── index.tsx
│   └── style.ts
├── Search/
│   ├── index.tsx
│   └── style.ts
├── CopyButton/
│   ├── index.tsx
│   └── style.ts
├── StatusDot/
│   ├── index.tsx
│   └── style.ts
├── Divider/
│   ├── index.tsx
│   └── style.ts
└── Skeleton/
    ├── index.tsx
    └── style.ts
```

---

## 2. 组件拆分

### base/ 层（20 个组件）

| 组件 | 说明 | 对应 Ant Design | 新增 Props |
|------|------|----------------|-----------|
| Button | 品牌化按钮 | Button | variant, size |
| Card | 品牌化卡片 | Card | variant, padding |
| Input | 品牌化输入框 | Input | variant |
| Tag | 语义化标签 | Tag | variant（多色） |
| Badge | 状态徽标 | Badge | variant |
| Empty | 空状态 | Empty | 接入 Token |
| Spinner | 加载动画 | Spin | 品牌色 |
| Avatar | 头像 | Avatar | 品牌回退色 |
| Tooltip | 提示 | Tooltip | variant |
| Modal | 弹窗 | Modal | 品牌圆角/阴影 |
| Drawer | 抽屉 | Drawer | 品牌圆角/阴影 |
| Table | 表格 | Table | compact/card 模式 |
| Tabs | 标签页 | Tabs | pill/underline 模式 |
| Alert | 警告提示 | Alert | brand variant |
| Form | 表单 | Form | gap 配置 |
| Search | 搜索框 | Input.Search | 胶囊圆角 |
| CopyButton | 复制按钮 | Button + Icon | 反馈状态 |
| StatusDot | 状态圆点 | 自定义 | pulse 动画 |
| Divider | 分割线 | Divider | variant |
| Skeleton | 骨架屏 | Skeleton | 多种变体 |

### 组件 Props 规范

```typescript
// ButtonProps
export interface ButtonProps extends AntButtonProps {
  variant?: 'primary' | 'secondary' | 'ghost' | 'danger' | 'link';
  size?: 'small' | 'medium' | 'large';
}

// CardProps
export interface CardProps extends AntCardProps {
  variant?: 'default' | 'elevated' | 'outlined';
  padding?: 'none' | 'small' | 'default' | 'large';
}

// TagProps
export interface TagProps extends AntTagProps {
  variant?: 'default' | 'success' | 'warning' | 'error' | 'info' | 'brand';
}
```

---

## 3. 数据流

无业务数据流。组件为纯展示型，数据通过 Props 传入。

```
父组件 ──► Props ──► Base Component ──► UI 渲染
```

---

## 4. 状态管理

- 组件内部状态：使用 `useState` 管理交互状态（如 CopyButton 的复制成功状态）
- 无全局状态依赖

---

## 5. API 调用

无 API 调用。

---

## 6. 用户交互流程

| 组件 | 交互 | 行为 | 结果 |
|------|------|------|------|
| Button | 点击 | onClick 回调 | 执行业务逻辑 |
| CopyButton | 点击 | 复制文本到剪贴板 | 显示成功反馈 |
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

---

## 8. 性能优化

- 组件使用 `React.memo` 避免不必要的重渲染
- 样式使用 CSS-in-JS 或 CSS Modules，避免全局污染
- 支持 Tree-shaking（按需导出）

---

## 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 与 Ant Design 样式冲突 | 中 | 二次封装可能覆盖 Ant Design 默认样式 | 使用 `ConfigProvider` 透传 Token |
| 组件 API 不兼容 | 中 | 新增 Props 可能与 Ant Design 未来版本冲突 | 使用 `extends` 继承原 Props |
| 暗色模式适配遗漏 | 中 | 新增组件可能遗漏暗色模式样式 | 建立组件审查清单 |

---

## 10. 开发步骤拆分

### Step 1: 核心组件（Button, Card, Input, Tag）（2 人日）
- [ ] Button 组件（含 variant/size）
- [ ] Card 组件（含 variant/padding）
- [ ] Input 组件（含 variant）
- [ ] Tag 组件（含多色 variant）

### Step 2: 状态组件（Badge, Empty, Spinner, Avatar, StatusDot）（1 人日）
- [ ] Badge 组件
- [ ] Empty 组件
- [ ] Spinner 组件
- [ ] Avatar 组件
- [ ] StatusDot 组件（含 pulse 动画）

### Step 3: 交互组件（Tooltip, Modal, Drawer, Tabs, Alert）（1 人日）
- [ ] Tooltip 组件
- [ ] Modal 组件
- [ ] Drawer 组件
- [ ] Tabs 组件（含 pill/underline 模式）
- [ ] Alert 组件

### Step 4: 表单组件（Form, Search, CopyButton, Divider, Skeleton）（1 人日）
- [ ] Form 组件
- [ ] Search 组件
- [ ] CopyButton 组件
- [ ] Divider 组件
- [ ] Skeleton 组件

### Step 5: 统一导出 & 文档（1 人日）
- [ ] 创建 `index.ts` 统一导出
- [ ] 编写组件使用文档
- [ ] 编写 Storybook  stories（可选）
