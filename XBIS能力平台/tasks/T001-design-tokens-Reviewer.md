# T001 Design Tokens 系统 — 设计评审报告

> 评审日期: 2026-04-24  
> 评审人: 技术负责人 + 架构审查官  
> 状态: Passed with changes

---

## 评审结果

### 状态: 👉 Passed with changes

---

## 问题列表

### 问题1: Token 命名规范不一致
**严重程度: 中**

任务卡中定义的 Token 命名（如 `fontSize: { xs: string }`）与设计方案中的命名（如 `font.size.xs`）存在不一致。开发时会产生歧义。

- 任务卡: `colors.brand.50` / `space.1`
- 设计方案: `colors.brand.default` / `space[0-20]`

### 问题2: 缺少 Z-Index Token 定义
**严重程度: 低**

未定义 z-index 层级 Token，但管理端弹窗、抽屉、下拉菜单等组件需要统一的层级管理。

### 问题3: 缺少 Transition/Animation Token
**严重程度: 低**

任务卡明确排除了动画 Tokens，但基础过渡效果（hover 过渡、主题切换过渡）是暗色模式体验的关键。

### 问题4: Breakpoints Token 与任务卡范围不一致
**严重程度: 低**

设计方案新增了 `breakpoints.ts`，但任务卡功能范围中未明确列出。

### 问题5: Semantic Token 暗色映射不完整
**严重程度: 中**

未提供具体的暗色模式映射值，仅定义了接口结构。

### 问题6: TokenProvider 分层归属不当
**严重程度: 低**

任务卡将 `TokenProvider` 标记为 `layout` 层，但应为应用级基础设施。

---

## 修改建议

### 建议1: 统一 Token 命名规范（必须修改）
统一采用点号命名法：
```typescript
colors.brand.50
colors.background.primary
space[0] ~ space[20]
font.size.xs ~ font.size.4xl
```

### 建议2: 补充 Z-Index Token（建议修改）
```typescript
zIndex: {
  base: 0, dropdown: 100, sticky: 200,
  fixed: 300, modalBackdrop: 400, modal: 500,
  popover: 600, tooltip: 700, toast: 800
}
```

### 建议3: 补充基础 Transition Token（建议修改）
```typescript
transition: {
  none: 'none', fast: '100ms',
  base: '200ms', slow: '300ms', theme: '200ms'
}
```

### 建议4: 明确 Breakpoints 范围（必须修改）
在任务卡功能范围中明确添加响应式断点 Tokens，或从设计方案中移除。

### 建议5: 补充完整暗色模式映射表（必须修改）
在 `themes/dark.ts` 中补充具体映射值。

### 建议6: TokenProvider 重新归类（建议修改）
移至 `packages/shared/src/theme/` 或作为独立包。

---

## 是否允许进入开发

### 👉 YES（附条件通过）

**必须完成的条件**:
1. 统一命名规范（建议1）
2. 明确 Breakpoints 范围（建议4）
3. 补充暗色模式映射表（建议5）

---

## 评审总结

| 检查项 | 结果 | 说明 |
|--------|------|------|
| 组件复用原则 | N/A | 无组件 |
| 页面模板使用 | N/A | 无页面 |
| 重复组件 | N/A | 无组件 |
| 分层规范 | ⚠️ | TokenProvider 分层需调整 |
| API 合理性 | N/A | 无 API |
| 状态设计完整性 | N/A | 无状态 |
| 能力平台架构 | ✅ | 符合 |
| 过度设计 | ⚠️ | Breakpoints 范围需明确 |
| 未声明数据模型 | ⚠️ | 缺少 z-index、transition |

**风险等级**: 低
