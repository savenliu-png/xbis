你现在扮演：

👉 **资深前端架构师 + 组件体系设计专家 + 文档维护负责人**

---

# 🎯 目标

基于当前代码实现，对组件架构文档进行：

# ❗同步更新（Document Sync）

生成：

👉 最新版本的 `docs/component-architecture.md`

---

# 📦 输入

1. 当前组件库代码结构（packages/components）
2. 当前任务卡（tasks/T002-base-components.md）
3. 当前验收报告（含新增组件/优化项）
4. 旧版 component-architecture.md

---

# 🔍 执行任务

---

## 1️⃣ 分析当前组件体系

必须识别：

* base 层组件（完整列表）
* business 层组件（如有）
* blocks 层组件（如有）
* layout 层结构

---

## 2️⃣ 更新组件清单（必须完整）

输出：

### Base Components

列出全部组件（包括新增）：

例如：

* Button
* Input
* Select（新增）
* Pagination（新增）

---

## 3️⃣ 更新组件分层规范

必须明确：

* base 层职责
* business 层职责
* blocks 层职责
* 禁止跨层调用规则

---

## 4️⃣ 更新组件设计原则

包括：

* 必须使用 Design Tokens
* 必须支持暗色模式
* 必须透传 Ant Design Props
* 禁止硬编码样式

---

## 5️⃣ 更新组件开发规范（必须）

包括：

* 文件结构规范
* 命名规范
* Props 设计规范
* 是否使用 React.memo
* 是否支持 variant / size

---

## 6️⃣ 更新组件使用规范（很关键）

必须明确：

* 页面必须优先使用 base 组件
* 禁止直接使用 Ant Design（除非特殊说明）
* 统一通过组件库导出

---

## 7️⃣ 标记新增/变更（可选）

在文档中标记：

* 新增组件（NEW）
* 已优化组件（UPDATED）

---

# 📊 输出结构

---

# 组件架构文档（component-architecture.md）

## 1. 架构概览

## 2. 组件分层

## 3. Base 组件清单（完整）

## 4. Business 组件说明

## 5. Blocks 组件说明

## 6. 设计原则

## 7. 开发规范

## 8. 使用规范

## 9. 注意事项

---

# ⚠️ 规则

* 不允许遗漏组件
* 不允许与当前代码不一致
* 必须覆盖新增组件
* 必须工程化表达
* 必须可作为后续开发标准
