# XBIS Design Tokens 设计规范

> 版本：v1.0  
> 适用范围：XBIS 用户端 + 管理端  
> 风格定位：SaaS + AI，简洁、可信、现代

---

## 1. 设计原则

### 1.1 品牌气质
- **简洁**：界面元素克制，留白充足，信息层级清晰
- **可信**：色彩稳重，交互反馈明确，数据展示专业
- **现代**：圆角柔和，阴影细腻，动效流畅

### 1.2 设计基础
- **基线网格**：4px Baseline Grid，所有间距、尺寸均为 4 的倍数
- **色彩系统**：提取自 `UI/styles.png` 的黄绿色品牌调，搭配中性灰与功能色
- **暗色模式**：全量 Token 支持 `light` / `dark` 双模式
- **跨端共享**：用户端与管理端共用同一套 Token，差异化通过组件级配置实现

---

## 2. 颜色 (Colors)

### 2.1 品牌主色

品牌主色提取自 `UI/styles.png` 的黄绿色调，传递可持续、自然、科技的混合气质。

| Token | 色值 (Light) | 色值 (Dark) | 用途 |
|-------|-------------|-------------|------|
| `brand.default` | `#6b8c2f` | `#8fb840` | 主按钮、品牌标识、关键链接 |
| `brand.hover` | `#5a7628` | `#a8c45f` | 按钮悬浮态 |
| `brand.active` | `#485f20` | `#c9db9e` | 按钮激活态 |
| `brand.subtle` | `#f4f7ec` | `#141a08` | 品牌背景、标签底色 |
| `brand.muted` | `#e3ebd0` | `#263110` | 品牌浅背景、hover 背景 |

### 2.2 功能色

| 类型 | Token | 色值 (Light) | 色值 (Dark) | 用途 |
|------|-------|-------------|-------------|------|
| 成功 | `success.default` | `#22c55e` | `#4ade80` | 成功状态、通过提示 |
| 成功 | `success.subtle` | `#f0fdf4` | `#052e16` | 成功背景 |
| 警告 | `warning.default` | `#f59e0b` | `#fbbf24` | 警告状态、待处理 |
| 警告 | `warning.subtle` | `#fffbeb` | `#451a03` | 警告背景 |
| 错误 | `error.default` | `#ef4444` | `#f87171` | 错误状态、删除操作 |
| 错误 | `error.subtle` | `#fef2f2` | `#450a0a` | 错误背景 |
| 信息 | `info.default` | `#3b82f6` | `#60a5fa` | 信息提示、AI 辅助 |
| 信息 | `info.subtle` | `#eff6ff` | `#172554` | 信息背景 |
| AI | `ai.default` | `#8b5cf6` | `#a78bfa` | AI 功能标识、智能标签 |
| AI | `ai.glow` | `rgba(139,92,246,0.25)` | `rgba(139,92,246,0.35)` | AI 元素发光效果 |

### 2.3 中性色

| Token | 色值 (Light) | 色值 (Dark) | 用途 |
|-------|-------------|-------------|------|
| `surface.default` | `#ffffff` | `#0d0f11` | 页面主背景 |
| `surface.subtle` | `#f8f9fa` | `#212529` | 次级背景、侧边栏 |
| `surface.muted` | `#f1f3f5` | `#343a40` | 卡片背景、分隔区域 |
| `surface.elevated` | `#ffffff` | `#343a40` | 悬浮卡片、弹窗背景 |
| `text.primary` | `#212529` | `#f8f9fa` | 主标题、正文 |
| `text.secondary` | `#868e96` | `#adb5bd` | 次要文字、描述 |
| `text.muted` | `#adb5bd` | `#868e96` | 禁用文字、占位符 |
| `text.inverse` | `#ffffff` | `#212529` | 深色背景上的文字 |
| `border.default` | `#e9ecef` | `#212529` | 默认边框 |
| `border.strong` | `#dee2e6` | `#343a40` | 强调边框 |

### 2.4 管理端专属色

| Token | 色值 (Light) | 色值 (Dark) | 用途 |
|-------|-------------|-------------|------|
| `admin.nav.bg` | `#212529` | `#0d0f11` | 管理端侧边栏背景 |
| `admin.nav.text` | `#adb5bd` | `#868e96` | 管理端导航文字 |
| `admin.nav.textActive` | `#ffffff` | `#ffffff` | 管理端导航激活文字 |
| `admin.nav.indicator` | `#8fb840` | `#8fb840` | 管理端导航指示器 |
| `admin.table.rowAlt` | `#f8f9fa` | `#212529` | 表格交替行背景 |
| `admin.table.rowHover` | `#f1f3f5` | `#343a40` | 表格行悬浮背景 |
| `admin.table.headerBg` | `#f1f3f5` | `#343a40` | 表格头部背景 |

---

## 3. 间距 (Spacing)

### 3.1 基础间距阶梯

基于 4px 基线网格：

| Token | 值 | 像素 | 典型场景 |
|-------|-----|------|----------|
| `space.0` | `0px` | 0 | 无间距 |
| `space.1` | `4px` | 4 | 图标内边距、紧凑标签 |
| `space.2` | `8px` | 8 | 按钮内图标间距、小间隙 |
| `space.3` | `12px` | 12 | 表单标签间隙、小卡片内边距 |
| `space.4` | `16px` | 16 | 标准组件内边距、卡片间隙 |
| `space.5` | `20px` | 20 | 表单项垂直间距 |
| `space.6` | `24px` | 24 | 页面内边距、卡片内边距 |
| `space.8` | `32px` | 32 | 模块间距、表单组间距 |
| `space.10` | `40px` | 40 | 大模块间距 |
| `space.12` | `48px` | 48 | Section 间距 |
| `space.16` | `64px` | 64 | 页面级间距 |
| `space.20` | `80px` | 80 | 大 Section 分隔 |

### 3.2 组件间距

| 组件 | Token | 值 | 说明 |
|------|-------|-----|------|
| 页面 | `componentSpace.page.paddingX` | `24px` | 页面水平内边距 |
| 页面 | `componentSpace.page.maxWidth` | `1280px` | 内容区最大宽度 |
| 卡片 | `componentSpace.card.padding` | `24px` | 标准卡片内边距 |
| 卡片 | `componentSpace.card.paddingCompact` | `16px` | 紧凑卡片 |
| 表单 | `componentSpace.form.rowGap` | `20px` | 表单项间距 |
| 按钮 | `componentSpace.button.paddingX` | `16px` | 按钮水平内边距 |
| 表格 | `componentSpace.table.cellPaddingX` | `16px` | 单元格水平内边距 |
| 表格 | `componentSpace.table.cellPaddingCompact` | `8px` | 紧凑表格 |
| 导航 | `componentSpace.nav.itemHeight` | `44px` | 导航项高度 |
| 模态框 | `componentSpace.modal.padding` | `24px` | 弹窗内边距 |

---

## 4. 字体 (Typography)

### 4.1 字体栈

| Token | 值 | 用途 |
|-------|-----|------|
| `fontFamily.sans` | `-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, "Noto Sans", "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", sans-serif` | 主字体（中西文） |
| `fontFamily.mono` | `"SF Mono", Monaco, Inconsolata, "Fira Code", "Roboto Mono", monospace` | 代码、密钥、ID |
| `fontFamily.numeric` | `"SF Pro Display", "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif` | 数字、统计数据 |

### 4.2 字号阶梯

| Token | 值 | 用途 |
|-------|-----|------|
| `fontSize.2xs` | `10px` | 时间戳、角标 |
| `fontSize.xs` | `12px` | 辅助文字、标签、表格小字 |
| `fontSize.sm` | `14px` | 正文次要、表格内容 |
| `fontSize.base` | `16px` | 正文主体 |
| `fontSize.lg` | `18px` | 卡片标题、强调正文 |
| `fontSize.xl` | `20px` | 模块标题 |
| `fontSize.2xl` | `24px` | 页面标题 (H3) |
| `fontSize.3xl` | `30px` | 页面主标题 (H2) |
| `fontSize.4xl` | `36px` | 营销标题 (H1) |
| `fontSize.5xl` | `48px` | Hero 区域 |

### 4.3 文本样式组合

| Token | 字号 | 字重 | 行高 | 用途 |
|-------|------|------|------|------|
| `textStyle.h1` | 36px | 700 | 1.25 | 页面主标题 |
| `textStyle.h2` | 30px | 600 | 1.25 | 页面标题 |
| `textStyle.h3` | 24px | 600 | 1.375 | 模块标题 |
| `textStyle.h4` | 20px | 600 | 1.375 | 卡片标题 |
| `textStyle.h5` | 18px | 500 | 1.5 | 小标题 |
| `textStyle.body` | 16px | 400 | 1.5 | 正文主体 |
| `textStyle.bodySmall` | 14px | 400 | 1.5 | 次要正文 |
| `textStyle.caption` | 12px | 400 | 1.5 | 辅助说明 |
| `textStyle.label` | 14px | 500 | 1 | 按钮、标签 |
| `textStyle.code` | 14px | 400 | 1.5 | 代码块 |
| `textStyle.numeric` | 24px | 600 | 1.25 | 统计数字 |
| `textStyle.numericLarge` | 36px | 700 | 1.25 | 大统计数字 |

---

## 5. 圆角 (Border Radius)

| Token | 值 | 用途 |
|-------|-----|------|
| `radius.none` | `0px` | 表格、分割线 |
| `radius.sm` | `4px` | 标签、小按钮 |
| `radius.base` | `6px` | 输入框、小卡片 |
| `radius.md` | `8px` | 按钮、卡片、弹窗 |
| `radius.lg` | `12px` | 大卡片、模块容器 |
| `radius.xl` | `16px` | 特色卡片、营销组件 |
| `radius.2xl` | `24px` | Hero 区域、品牌元素 |
| `radius.full` | `9999px` | 胶囊按钮、圆形头像 |

### 组件级圆角

| 组件 | Token | 值 |
|------|-------|-----|
| 按钮 | `componentRadius.button.default` | `8px` |
| 按钮(胶囊) | `componentRadius.button.pill` | `9999px` |
| 输入框 | `componentRadius.input.default` | `6px` |
| 卡片 | `componentRadius.card.default` | `12px` |
| 卡片(特色) | `componentRadius.card.featured` | `16px` |
| 弹窗 | `componentRadius.modal.default` | `12px` |
| 标签 | `componentRadius.tag.default` | `4px` |
| 头像 | `componentRadius.image.avatar` | `9999px` |

---

## 6. 阴影 (Shadows)

### 6.1 层级阴影 (Elevation)

| Token | 值 (Light) | 值 (Dark) | 用途 |
|-------|-----------|-----------|------|
| `elevation.0` | `none` | `none` | 无阴影 |
| `elevation.1` | `0 1px 2px rgba(0,0,0,0.04)` | `0 1px 2px rgba(0,0,0,0.15)` | 内嵌元素 |
| `elevation.2` | `0 1px 3px rgba(0,0,0,0.08), 0 1px 2px rgba(0,0,0,0.12)` | `0 1px 3px rgba(0,0,0,0.25), 0 1px 2px rgba(0,0,0,0.30)` | 卡片默认 |
| `elevation.3` | `0 4px 6px rgba(0,0,0,0.08), 0 2px 4px rgba(0,0,0,0.12)` | `0 4px 6px rgba(0,0,0,0.25), 0 2px 4px rgba(0,0,0,0.30)` | 悬浮卡片、下拉菜单 |
| `elevation.4` | `0 10px 15px rgba(0,0,0,0.08), 0 4px 6px rgba(0,0,0,0.12)` | `0 10px 15px rgba(0,0,0,0.25), 0 4px 6px rgba(0,0,0,0.30)` | 模态框、抽屉 |
| `elevation.5` | `0 20px 25px rgba(0,0,0,0.08), 0 8px 10px rgba(0,0,0,0.12)` | `0 20px 25px rgba(0,0,0,0.25), 0 8px 10px rgba(0,0,0,0.30)` | 全屏弹窗 |

### 6.2 状态阴影

| Token | 值 (Light) | 值 (Dark) | 用途 |
|-------|-----------|-----------|------|
| `stateShadow.focus` | `0 0 0 3px rgba(59,130,246,0.25)` | `0 0 0 3px rgba(59,130,246,0.35)` | 聚焦环 |
| `stateShadow.focusBrand` | `0 0 0 3px rgba(107,140,47,0.25)` | `0 0 0 3px rgba(107,140,47,0.35)` | 品牌聚焦 |
| `stateShadow.hover` | `0 8px 16px rgba(0,0,0,0.08)` | `0 8px 16px rgba(0,0,0,0.25)` | 悬浮提升 |
| `stateShadow.aiGlow` | `0 0 20px rgba(139,92,246,0.25)` | `0 0 24px rgba(139,92,246,0.35)` | AI 发光 |
| `stateShadow.brandGlow` | `0 0 16px rgba(107,140,47,0.15)` | `0 0 20px rgba(107,140,47,0.25)` | 品牌发光 |

---

## 7. 用户端 & 管理端 Token 复用关系

### 7.1 完全共享 (100% 复用)

以下 Token 用户端与管理端完全一致，直接共享：

- `primitiveColors` — 原始色板
- `semanticColors.surface` — 背景色
- `semanticColors.text` — 文字色
- `semanticColors.border` — 边框色
- `semanticColors.brand` — 品牌色
- `semanticColors.success/warning/error/info` — 状态色
- `space` — 基础间距
- `componentSpace` — 组件间距（管理端表格默认使用紧凑模式）
- `layout` — 布局参数（断点、栅格、侧边栏）
- `fontFamily/fontSize/fontWeight/lineHeight/letterSpacing` — 字体属性
- `radius` — 圆角
- `elevation/stateShadow` — 阴影

### 7.2 差异化使用

| Token | 用户端 | 管理端 | 差异说明 |
|-------|--------|--------|----------|
| `textStyle.hero/5xl/6xl` | 营销页使用 | 不使用 | 管理端保持专业克制 |
| `componentSpace.table.cellPadding` | 标准模式 | 紧凑模式 | 管理端信息密度更高 |
| `componentRadius.card.featured` | 高频使用 | 低频使用 | 用户端强调品牌亲和力 |
| `adminAccent.nav` | 不使用 | 独占 | 管理端深色导航专属 |
| `adminAccent.table` | 不使用 | 独占 | 管理端表格样式专属 |
| `semanticColors.ai` | 高频 (AI 功能) | 低频 | 管理端仅在智能模块使用 |

---

## 8. CSS / Tailwind / SCSS 映射建议

### 8.1 纯 CSS 变量映射

```css
:root {
  /* Primitive */
  --color-green-500: #6b8c2f;
  --color-gray-100: #f1f3f5;
  --color-gray-900: #212529;

  /* Semantic (Light) */
  --surface-default: #ffffff;
  --surface-subtle: #f8f9fa;
  --text-primary: #212529;
  --text-secondary: #868e96;
  --border-default: #e9ecef;
  --brand-default: #6b8c2f;
  --brand-hover: #5a7628;
  --success-default: #22c55e;
  --warning-default: #f59e0b;
  --error-default: #ef4444;

  /* Spacing */
  --space-1: 4px;
  --space-2: 8px;
  --space-4: 16px;
  --space-6: 24px;

  /* Radius */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;

  /* Shadow */
  --shadow-elevation-2: 0 1px 3px rgba(0,0,0,0.08), 0 1px 2px rgba(0,0,0,0.12);
  --shadow-focus: 0 0 0 3px rgba(59,130,246,0.25);
}

[data-theme="dark"] {
  --surface-default: #0d0f11;
  --surface-subtle: #212529;
  --text-primary: #f8f9fa;
  --text-secondary: #adb5bd;
  --border-default: #212529;
  --brand-default: #8fb840;
  --brand-hover: #a8c45f;
}
```

### 8.2 Tailwind CSS 配置映射

```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        brand: {
          DEFAULT: '#6b8c2f',
          hover: '#5a7628',
          active: '#485f20',
          subtle: '#f4f7ec',
          muted: '#e3ebd0',
        },
        surface: {
          DEFAULT: '#ffffff',
          subtle: '#f8f9fa',
          muted: '#f1f3f5',
          elevated: '#ffffff',
        },
        // ... 其他语义色
      },
      spacing: {
        'space-1': '4px',
        'space-2': '8px',
        'space-3': '12px',
        'space-4': '16px',
        'space-5': '20px',
        'space-6': '24px',
        'space-8': '32px',
        'space-10': '40px',
        'space-12': '48px',
        'space-16': '64px',
      },
      borderRadius: {
        'sm': '4px',
        'base': '6px',
        'md': '8px',
        'lg': '12px',
        'xl': '16px',
        '2xl': '24px',
      },
      boxShadow: {
        'elevation-1': '0 1px 2px 0 rgba(0,0,0,0.04)',
        'elevation-2': '0 1px 3px 0 rgba(0,0,0,0.08), 0 1px 2px -1px rgba(0,0,0,0.12)',
        'elevation-3': '0 4px 6px -1px rgba(0,0,0,0.08), 0 2px 4px -2px rgba(0,0,0,0.12)',
        'focus': '0 0 0 3px rgba(59,130,246,0.25)',
        'focus-brand': '0 0 0 3px rgba(107,140,47,0.25)',
      },
      fontFamily: {
        sans: ['-apple-system', 'BlinkMacSystemFont', 'Segoe UI', 'Roboto', 'Helvetica Neue', 'Arial', 'Noto Sans', 'PingFang SC', 'Microsoft YaHei', 'sans-serif'],
        mono: ['SF Mono', 'Monaco', 'Inconsolata', 'Fira Code', 'monospace'],
      },
      fontSize: {
        '2xs': ['10px', { lineHeight: '1.5' }],
        'xs': ['12px', { lineHeight: '1.5' }],
        'sm': ['14px', { lineHeight: '1.5' }],
        'base': ['16px', { lineHeight: '1.5' }],
        'lg': ['18px', { lineHeight: '1.5' }],
        'xl': ['20px', { lineHeight: '1.375' }],
        '2xl': ['24px', { lineHeight: '1.25' }],
        '3xl': ['30px', { lineHeight: '1.25' }],
        '4xl': ['36px', { lineHeight: '1.25' }],
        '5xl': ['48px', { lineHeight: '1.25' }],
      },
    },
  },
};
```

### 8.3 SCSS 变量映射

```scss
// _tokens.scss

// Primitive
$color-green-500: #6b8c2f;
$color-green-600: #5a7628;
$color-gray-0: #ffffff;
$color-gray-50: #f8f9fa;
$color-gray-100: #f1f3f5;
$color-gray-900: #212529;
$color-gray-950: #0d0f11;

// Semantic (Light)
$surface-default: $color-gray-0;
$surface-subtle: $color-gray-50;
$text-primary: $color-gray-900;
$text-secondary: #868e96;
$border-default: #e9ecef;
$brand-default: $color-green-500;
$brand-hover: $color-green-600;

// Spacing
$space-1: 4px;
$space-2: 8px;
$space-3: 12px;
$space-4: 16px;
$space-6: 24px;
$space-8: 32px;

// Radius
$radius-sm: 4px;
$radius-md: 8px;
$radius-lg: 12px;

// Shadow
$shadow-elevation-2: 0 1px 3px rgba(0,0,0,0.08), 0 1px 2px rgba(0,0,0,0.12);
$shadow-focus: 0 0 0 3px rgba(59,130,246,0.25);

// Dark mode mixin
@mixin dark-mode {
  @media (prefers-color-scheme: dark) {
    @content;
  }
  [data-theme="dark"] & {
    @content;
  }
}
```

---

## 9. 与 Ant Design 5.x 的映射

本项目使用 Ant Design 5.x，可通过 `ConfigProvider` 将 Design Tokens 映射到 Ant Design 组件：

```tsx
import { ConfigProvider, theme as antdTheme } from 'antd';
import { colors, radius, shadows } from '@xbis/tokens';

const customTheme = {
  token: {
    colorPrimary: colors.brand.default,
    colorSuccess: colors.success.default,
    colorWarning: colors.warning.default,
    colorError: colors.error.default,
    colorInfo: colors.info.default,
    borderRadius: parseInt(radius.md),
    fontFamily: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, "Noto Sans", sans-serif',
  },
  algorithm: antdTheme.defaultAlgorithm, // 或 darkAlgorithm
};

// 使用
<ConfigProvider theme={customTheme}>
  <App />
</ConfigProvider>
```

---

## 10. 文件清单

| 文件 | 说明 |
|------|------|
| `tokens/colors.ts` | 颜色 Token（原始色 + 语义色 + 暗色模式 + 管理端专属） |
| `tokens/spacing.ts` | 间距 Token（基础间距 + 组件间距 + 布局网格） |
| `tokens/typography.ts` | 字体 Token（字体栈 + 字号 + 字重 + 行高 + 文本样式组合） |
| `tokens/radius.ts` | 圆角 Token（基础圆角 + 组件级圆角） |
| `tokens/shadows.ts` | 阴影 Token（层级阴影 + 状态阴影 + 组件阴影 + 暗色模式） |
| `tokens/index.ts` | 统一导出入口 |
| `docs/design-tokens.md` | 本设计规范文档 |

---

## 11. 使用示例

```tsx
import { colors, space, textStyle, radius, shadows } from '@xbis/tokens';

// 在 styled-components / emotion 中使用
const Card = styled.div`
  background: ${colors.surface.elevated};
  padding: ${space['6']};
  border-radius: ${radius.lg};
  box-shadow: ${shadows.elevation[2]};
`;

// 在普通 CSS-in-JS 中使用
const titleStyle = {
  ...textStyle.h3,
  color: colors.text.primary,
};

// 暗色模式切换
import { getColors, getShadows } from '@xbis/tokens';
const darkColors = getColors('dark');
const darkShadows = getShadows('dark');
```

---

*本文档与 Token 文件同步维护，后续迭代请同步更新。*
