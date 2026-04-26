# T001 Design Tokens 系统 — 修正版设计方案（V2）

> 任务卡: [T001-design-tokens.md](../T001-design-tokens.md)  
> 设计输入: docs/design-tokens.md, docs/component-architecture.md, docs/dev-rules.md  
> 原设计方案: [T001-design-tokens-spec.md](../T001-design-tokens-spec.md)  
> 评审报告: [T001-design-tokens-Reviewer.md](../T001-design-tokens-Reviewer.md)  
> 输出日期: 2026-04-24  
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed with changes

---

## 二、必须修改项（Blocking）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | Token 命名规范不一致 | 附录：Token 命名规范 |
| 2 | Breakpoints Token 范围未明确 | 第 2 节组件拆分 |
| 3 | Semantic Token 暗色映射不完整 | 附录：暗色模式映射表 |

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 缺少 Z-Index Token | 附录：新增 z-index.ts |
| 2 | 缺少 Transition Token | 附录：新增 transitions.ts |
| 3 | TokenProvider 分层归属 | 第 2 节组件拆分 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

本任务为**基础能力层**建设，无页面结构。输出物为 Design Tokens 代码包 + 文档。

```
packages/tokens/                    # Token 包（已存在，需补全）
├── colors.ts                       # 颜色 Token
├── spacing.ts                      # 间距 Token
├── typography.ts                   # 字体 Token
├── radius.ts                       # 圆角 Token
├── shadows.ts                      # 阴影 Token
├── breakpoints.ts                  # 响应式断点 Token 【已修复：明确纳入范围】
├── z-index.ts                      # 层级 Token 【新增】
├── transitions.ts                  # 过渡 Token 【新增】
├── index.ts                        # 统一导出
└── themes/                         # 主题配置
    ├── light.ts
    └── dark.ts                     # 【已修复：补充完整映射表】
```

---

### 2. 组件拆分

#### base/ 层（无 UI 组件，纯 Token 变量）

| Token 类别 | 文件 | 说明 |
|-----------|------|------|
| Colors | `tokens/colors.ts` | 品牌色/功能色/中性色/暗色模式映射 |
| Spacing | `tokens/spacing.ts` | 4px 基数 scale |
| Typography | `tokens/typography.ts` | 字体栈/字号/字重/行高 |
| Radius | `tokens/radius.ts` | 圆角 scale |
| Shadows | `tokens/shadows.ts` | 阴影层级 |
| Breakpoints | `tokens/breakpoints.ts` | 响应式断点 【已修复：明确为必做项】 |
| Z-Index | `tokens/z-index.ts` | 层级管理 【新增】 |
| Transitions | `tokens/transitions.ts` | 过渡动画 【新增】 |

#### 复用关系

- 用户端 & 管理端 **共用同一套 Token**，通过 `theme` 参数区分表现
- 管理端 Sidebar 使用深色背景，通过 `colors.background.inverse` 实现
- TokenProvider 归属 【已修复：移至 `packages/shared/src/theme/TokenProvider.tsx`，作为应用级基础设施】

---

### 3. 数据流

无运行时数据流。Token 为编译时静态导出。

```
设计规范 ──► Token 定义 ──► 组件引用 ──► 运行时样式
```

---

### 4. 状态管理

无状态管理需求。

---

### 5. API 调用

无 API 调用。

---

### 6. 用户交互流程

无用户交互。

---

### 7. 边界与异常

| 场景 | 处理方案 |
|------|----------|
| Token 值缺失 | 使用 fallback 值（Ant Design 默认） |
| 暗色模式切换 | 通过 `data-theme="dark"` 属性切换 CSS 变量 |
| 不支持的浏览器 | 提供 CSS 变量 + SCSS 变量双方案 |

---

### 8. 性能优化

- Token 为静态常量，无运行时计算开销
- CSS 变量方案支持浏览器缓存
- Tree-shaking 友好（按需导出）

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 暗色模式覆盖不全 | 中 | 新增组件可能遗漏暗色适配 | 建立 Token 使用检查清单 |
| 与 Ant Design Token 冲突 | 中 | 自定义 Token 可能覆盖 Ant Design 默认行为 | 使用 `ConfigProvider` 透传 |
| 多端表现不一致 | 低 | 用户端/管理端共用 Token 可能导致意外表现 | 通过 `theme` 参数区分 |

---

### 10. 开发步骤拆分

#### Step 1: 颜色 Token 补全（1 人日）
- [ ] 定义品牌色 9 级色阶（default/light-1~5/dark-1~3）
- [ ] 定义功能色（success/warning/error/info）
- [ ] 定义中性色（灰阶 1~12）
- [ ] 定义暗色模式映射表 【已修复：补充完整映射值】

#### Step 2: 间距/圆角/阴影 Token（0.5 人日）
- [ ] 定义 spacing scale（4px 基数，0~20）
- [ ] 定义 radius scale（0~16px）
- [ ] 定义阴影层级（none/small/medium/large/xl）

#### Step 3: 字体 Token（0.5 人日）
- [ ] 定义字体栈（中文优先）
- [ ] 定义字号 scale（xs~4xl）
- [ ] 定义字重（light/regular/medium/bold）
- [ ] 定义行高（tight/normal/relaxed）

#### Step 4: 断点 Token（0.5 人日）
- [ ] 定义响应式断点（xs~xxl） 【已修复：明确为必做项】

#### Step 5: 新增 Token 类型（0.5 人日）
- [ ] 定义 z-index 层级 Token 【新增】
- [ ] 定义 transition 过渡 Token 【新增】

#### Step 6: 主题配置（0.5 人日）
- [ ] 创建 light.ts 主题配置
- [ ] 创建 dark.ts 主题配置 【已修复：补充完整映射】
- [ ] 导出 CSS 变量映射

---

## 附录 A: Token 命名规范（V2）【已修复】

```typescript
// 颜色
 colors.brand.50          // 【已修复：统一为数字索引】
 colors.brand.100
 colors.brand.200
 colors.brand.300
 colors.brand.400
 colors.brand.500         // default
 colors.brand.600
 colors.brand.700
 colors.brand.800
 colors.brand.900
 colors.success.default
 colors.text.primary
 colors.background.default
 colors.border.default

// 间距
 space[0] ~ space[20]   // 0, 4, 8, 12, 16, 20, 24, 28, 32, 36, 40, 44, 48, 56, 64, 72, 80, 96, 120, 160

// 字体
 font.size.xs    // 12px
 font.size.sm    // 14px
 font.size.md    // 14px 【已修复：default 改为 md，避免与 JS 关键字冲突】
 font.size.lg    // 16px
 font.size.xl    // 20px
 font.size.2xl   // 24px
 font.size.3xl   // 32px
 font.size.4xl   // 48px

// 圆角
 radius.none    // 0
 radius.sm      // 4px
 radius.md      // 8px 【已修复：default 改为 md】
 radius.lg      // 12px
 radius.xl      // 16px
 radius.full    // 9999px

// 阴影
 shadow.none
 shadow.sm
 shadow.md      // 【已修复：default 改为 md】
 shadow.lg
 shadow.xl
```

---

## 附录 B: 暗色模式映射表（V2）【已修复】

```typescript
// themes/dark.ts
export const darkTheme = {
  colors: {
    background: {
      primary: colors.neutral[900],    // #1f1f1f
      secondary: colors.neutral[800],  // #2f2f2f
      tertiary: colors.neutral[700],   // #3f3f3f
      elevated: colors.neutral[800],   // #2f2f2f
      overlay: 'rgba(0, 0, 0, 0.8)',
    },
    text: {
      primary: colors.neutral[0],      // #ffffff
      secondary: colors.neutral[300],  // #d1d1d1
      tertiary: colors.neutral[500],   // #8c8c8c
      inverse: colors.neutral[900],    // #1f1f1f
      disabled: colors.neutral[600],   // #5f5f5f
    },
    border: {
      default: colors.neutral[700],    // #3f3f3f
      hover: colors.neutral[600],      // #5f5f5f
      active: colors.brand[400],       // 品牌色
      error: colors.error[500],        // 错误色
    },
  },
};
```

---

## 附录 C: Z-Index Token（V2）【新增】

```typescript
// tokens/z-index.ts
export const zIndex = {
  base: 0,
  dropdown: 100,
  sticky: 200,
  fixed: 300,
  modalBackdrop: 400,
  modal: 500,
  popover: 600,
  tooltip: 700,
  toast: 800,
};
```

---

## 附录 D: Transition Token（V2）【新增】

```typescript
// tokens/transitions.ts
export const transition = {
  none: 'none',
  fast: '100ms ease-in-out',
  base: '200ms ease-in-out',
  slow: '300ms ease-in-out',
  theme: '200ms color, 200ms background-color',
};
```

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **Token 命名** | `colors.brand.default` / `space.1` | `colors.brand.50~900` / `space[0~20]` |
| **Breakpoints** | 新增文件，范围未明确 | 明确纳入功能范围，必做项 |
| **暗色映射** | 仅接口结构 | 补充完整映射值 |
| **Z-Index** | 未定义 | 新增 `tokens/z-index.ts` |
| **Transition** | 未定义 | 新增 `tokens/transitions.ts` |
| **TokenProvider** | 归属 `layout` 层 | 移至 `packages/shared/src/theme/` |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（命名规范统一、Breakpoints 范围明确、暗色映射完整）
2. 新增 Token（z-index、transition）已补充
3. TokenProvider 分层已调整
4. 风险等级：低
