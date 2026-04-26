# T002 基础组件库 — 设计评审报告

> 评审日期: 2026-04-24  
> 评审人: 技术负责人 + 架构审查官  
> 状态: Passed with changes

---

## 评审结果

### 状态: 👉 Passed with changes

---

## 问题列表

### 问题1: 组件列表与 Ant Design 重叠度高，差异化不足
**严重程度: 中**

20 个组件中大部分为 Ant Design 的直接二次封装，差异化价值不明确。如 Input、Tag、Badge、Modal 等组件的新增 Props 较少。

**影响**: 开发成本与收益不成正比，可能导致开发者直接使用 Ant Design 组件绕过封装。

### 问题2: 缺少 Select / DatePicker / TimePicker 组件
**严重程度: 中**

基础表单组件中缺少 Select、DatePicker、TimePicker、Switch 等常用组件，后续业务组件和页面开发将被迫直接使用 Ant Design。

### 问题3: CopyButton 应属于 business 层而非 base 层
**严重程度: 低**

CopyButton 包含剪贴板操作和反馈状态，属于业务逻辑组件，不应放在 base 层。

### 问题4: StatusDot 与 Badge 功能重叠
**严重程度: 低**

StatusDot 和 Badge 都用于状态展示，可能存在功能重叠。建议明确区分：Badge 用于徽标计数，StatusDot 用于状态指示。

### 问题5: 未明确样式方案（CSS-in-JS vs CSS Modules）
**严重程度: 中**

设计方案提到 "样式使用 CSS-in-JS 或 CSS Modules"，但未明确选择。这会导致开发时风格不一致。

---

## 修改建议

### 建议1: 明确二次封装的价值点（必须修改）
每个组件必须明确说明：
- 与 Ant Design 原组件的差异点
- 新增的 Props 和默认值
- 为什么要封装（如统一品牌色、统一圆角）

### 建议2: 补充缺失的基础表单组件（必须修改）
增加以下组件到 base 层：
- Select / Select.Option
- DatePicker / RangePicker
- Switch
- Checkbox / Radio
- Upload

### 建议3: 将 CopyButton 移至 business 层（建议修改）
或将其降级为 base 层的纯 UI 组件（仅复制图标按钮，不含剪贴板逻辑）。

### 建议4: 明确 StatusDot 与 Badge 的边界（建议修改）
- Badge: 徽标、计数、红点通知
- StatusDot: 状态指示（在线/离线/处理中），支持 pulse 动画

### 建议5: 确定样式方案（必须修改）
统一使用 CSS Modules 方案，理由：
- 与 Ant Design 的 CSS-in-JS 解耦
- 更好的 Tree-shaking 支持
- 更易于主题切换

---

## 是否允许进入开发

### 👉 YES（附条件通过）

**必须完成的条件**:
1. 补充 Select、DatePicker、Switch 等缺失的基础组件（建议2）
2. 确定并统一样式方案为 CSS Modules（建议5）
3. 明确每个组件的封装价值点（建议1）

---

## 评审总结

| 检查项 | 结果 | 说明 |
|--------|------|------|
| 组件复用原则 | ✅ | 基于 Ant Design 复用 |
| 页面模板使用 | N/A | 无页面 |
| 重复组件 | ⚠️ | StatusDot/Badge 边界模糊 |
| 分层规范 | ⚠️ | CopyButton 分层不当 |
| API 合理性 | N/A | 无 API |
| 状态设计完整性 | ✅ | 纯展示组件 |
| 能力平台架构 | ✅ | 符合 |
| 过度设计 | ⚠️ | 部分组件封装价值不足 |
| 未声明数据模型 | ⚠️ | 缺少基础表单组件 |

**风险等级**: 中
