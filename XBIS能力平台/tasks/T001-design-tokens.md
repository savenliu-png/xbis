# Design Tokens 系统

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | Design Tokens 系统 |
| 任务编号 | T001 |
| 所属模块 ⭐ | M1 基础设计系统 |
| 优先级 | P0 |
| 指派给 | @frontend-architect |
| 预计工期 | 2 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-04-25 |

## 2. 需求背景

### 2.1 问题描述
当前项目缺乏统一的设计语言，颜色、字体、间距等样式散落在各组件中，导致：
- 用户端和管理端视觉不一致
- 暗色模式难以实现
- 样式维护成本高，修改一处需改多处

### 2.2 目标用户
- 前端开发者：使用统一的 Design Tokens 构建页面
- UI 设计师：通过 Tokens 控制全局视觉风格
- 未来维护者：快速调整全局样式

### 2.3 预期效果
- 建立完整的 Design Tokens 体系，覆盖颜色、字体、间距、圆角、阴影
- 支持暗色模式，通过切换 Token 实现主题变换
- 用户端和管理端共用一套 Tokens，通过语义化区分

## 3. 页面位置 ⭐

### 3.1 用户端
| 页面 | 路由 | 入口 |
|------|------|------|
| 全局样式 | — | 所有页面 |

### 3.2 管理端
| 页面 | 路由 | 入口 |
|------|------|------|
| 全局样式 | — | 所有页面 |

### 3.3 页面原型/设计稿
- 参考：`docs/design-tokens.md`
- 样式参考：`UI/styles.png`

## 4. 功能范围

### 4.1 包含功能
- [ ] 颜色 Tokens（primitive + semantic，支持 light/dark）
- [ ] 字体 Tokens（font family、size、weight、line height）
- [ ] 间距 Tokens（spacing scale、component spacing）
- [ ] 圆角 Tokens（radius scale）
- [ ] 阴影 Tokens（shadow scale）
- [ ] CSS / Tailwind / SCSS 映射配置

### 4.2 不包含功能（明确排除）
- 主题切换 UI（由具体页面实现）
- 动画 Tokens（V2 迭代）
- 图标 Tokens（使用现有图标库）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 颜色 Token（Primitive）
export interface ColorPrimitiveTokens {
  brand: {
    50: string;
    100: string;
    200: string;
    300: string;
    400: string;
    500: string;
    600: string;
    700: string;
    800: string;
    900: string;
  };
  neutral: {
    0: string;
    50: string;
    100: string;
    200: string;
    300: string;
    400: string;
    500: string;
    600: string;
    700: string;
    800: string;
    900: string;
    1000: string;
  };
  success: { 50: string; 100: string; 500: string; 700: string; 900: string };
  warning: { 50: string; 100: string; 500: string; 700: string; 900: string };
  error: { 50: string; 100: string; 500: string; 700: string; 900: string };
  info: { 50: string; 100: string; 500: string; 700: string; 900: string };
}

// 颜色 Token（Semantic）
export interface ColorSemanticTokens {
  background: {
    primary: string;
    secondary: string;
    tertiary: string;
    elevated: string;
    overlay: string;
  };
  text: {
    primary: string;
    secondary: string;
    tertiary: string;
    inverse: string;
    disabled: string;
  };
  border: {
    default: string;
    hover: string;
    active: string;
    error: string;
  };
}

// 字体 Token
export interface TypographyTokens {
  fontFamily: {
    sans: string;
    mono: string;
  };
  fontSize: {
    xs: string;
    sm: string;
    base: string;
    lg: string;
    xl: string;
    '2xl': string;
    '3xl': string;
    '4xl': string;
  };
  fontWeight: {
    normal: number;
    medium: number;
    semibold: number;
    bold: number;
  };
  lineHeight: {
    tight: number;
    normal: number;
    relaxed: number;
  };
}

// 间距 Token
export interface SpacingTokens {
  space: {
    0: string;
    1: string;
    2: string;
    3: string;
    4: string;
    5: string;
    6: string;
    8: string;
    10: string;
    12: string;
    16: string;
    20: string;
    24: string;
    32: string;
    40: string;
    48: string;
    64: string;
  };
}

// 圆角 Token
export interface RadiusTokens {
  radius: {
    none: string;
    sm: string;
    base: string;
    md: string;
    lg: string;
    xl: string;
    '2xl': string;
    full: string;
  };
}

// 阴影 Token
export interface ShadowTokens {
  shadow: {
    sm: string;
    base: string;
    md: string;
    lg: string;
    xl: string;
    '2xl': string;
    inner: string;
    none: string;
  };
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`tokens/colors.ts`, `tokens/typography.ts`, `tokens/spacing.ts`, `tokens/radius.ts`, `tokens/shadows.ts`
- 命名：`ColorTokens`, `TypographyTokens`, `SpacingTokens`, `RadiusTokens`, `ShadowTokens`

## 6. 交互流程

### 6.1 主流程
```
[开发者引入 Tokens] ──► [使用 Token 构建样式] ──► [主题切换时自动更新]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| Token 缺失 | 使用了未定义的 Token | 构建报错 | 开发时即发现 |
| 暗色模式切换 | 用户切换主题 | 所有使用 Token 的组件自动更新 | 无感知切换 |

### 6.3 边界情况
- 不支持 CSS 变量浏览器：提供 fallback
- 服务端渲染：确保 Tokens 在服务端可用

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
无

### 7.2 业务组件（business/）
无

### 7.3 页面块组件（blocks/）
无

### 7.4 页面模板（layout/）
无

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| TokenProvider | layout | 主题上下文提供者 |

## 8. API 接口 ⭐

无

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] 颜色 Tokens 覆盖 primitive（brand/neutral/success/warning/error/info）和 semantic（background/text/border）
- [ ] 字体 Tokens 覆盖 family/size/weight/lineHeight
- [ ] 间距 Tokens 覆盖 spacing scale（0-64）
- [ ] 圆角 Tokens 覆盖 radius scale（none-full）
- [ ] 阴影 Tokens 覆盖 shadow scale（sm-2xl）
- [ ] 支持 light/dark 两种主题
- [ ] 提供 CSS 变量映射
- [ ] 提供 Tailwind 配置映射
- [ ] 提供 SCSS 变量映射

### 9.2 技术验收
- [ ] TypeScript 类型完整，无 `any`
- [ ] 所有 Token 有明确的命名规范
- [ ] Token 值使用 CSS 变量或具体值
- [ ] 主题切换无闪烁

### 9.3 性能验收
- [ ] Token 文件体积 < 50KB（gzip）
- [ ] 主题切换 < 100ms

### 9.4 兼容性验收
- [ ] 支持 Chrome/Firefox/Safari/Edge 最新 2 个版本
- [ ] 不支持 IE

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 全新功能，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **否** — 可独立开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 暗色模式颜色对比度不足 | 中 | 高 | 使用对比度检查工具验证 |
| Token 命名冲突 | 低 | 中 | 使用 BEM-like 命名规范 |

## 11. 开发检查清单

### 11.1 开发前
- [x] 需求已评审并确认
- [x] 设计稿已确认
- [x] 数据结构和 API 已评审
- [x] 依赖任务已完成

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

- 参考已有文件：`tokens/colors.ts`, `tokens/typography.ts`, `tokens/spacing.ts`, `tokens/radius.ts`, `tokens/shadows.ts`
- 用户端和管理端共用 primitive Tokens，semantic Tokens 可区分

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
