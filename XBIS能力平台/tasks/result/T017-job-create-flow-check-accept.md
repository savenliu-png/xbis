# T017 任务创建流程 — 验收文档

## 1. 修改文件清单

| 序号 | 文件路径 | 修改类型 |
|------|----------|----------|
| 1 | `packages/user/src/App.tsx` | 修改 |
| 2 | `packages/user/src/pages/JobCreate.tsx` | 新增 |
| 3 | `packages/shared/src/constants/index.ts` | 修改 |
| 4 | `packages/shared/src/api/services.ts` | 修改 |
| 5 | `packages/components/business/AbilitySelector/index.tsx` | 新增 |
| 6 | `packages/components/business/ExecutorSelector/index.tsx` | 新增 |
| 7 | `packages/components/business/PayloadEditor/index.tsx` | 新增 |
| 8 | `packages/components/business/index.ts` | 修改 |
| 9 | `packages/components/blocks/JobCreateForm/index.tsx` | 新增 |
| 10 | `packages/components/blocks/index.ts` | 修改 |

## 2. 新增组件清单

| 组件名称 | 所属层级 | 用途 |
|----------|----------|------|
| AbilitySelector | business | 能力选择器（搜索 + 单选） |
| ExecutorSelector | business | 执行器选择器（状态展示 + 单选） |
| PayloadEditor | business | Payload 编辑器（Schema表单 / JSON模式切换） |
| JobCreateForm | blocks | 任务创建表单（组合上述组件） |

## 3. API变更清单

| API路径 | 调用位置 | 是否通过 Business Services |
|---------|----------|--------------------------|
| `GET /api/v1/abilities` | JobCreate.tsx | 是（userApi.abilities.list） |
| `GET /api/v1/abilities/:id/schema` | JobCreate.tsx | 是（userApi.abilities.getSchema） |
| `GET /api/v1/executors` | JobCreate.tsx | 是（userApi.executors.list） |
| `POST /api/v1/jobs` | JobCreate.tsx | 是（userApi.jobs.create） |

## 4. 风险说明

| 风险项 | 说明 |
|--------|------|
| 是否影响现有功能 | 否，新增页面 |
| 是否影响数据结构 | 否，使用已有 JobCreateRequest 类型 |
| 是否影响路由 | 是，新增 `/jobs/create` 路由 |
| 是否影响 Service 层 | 是，用户端 `jobs` 启用 `includeCreate: true`，`abilities` 新增 `getSchema`，新增 `executors` API |

## 5. 自检结果

### C5 AI 强制自检

| 问题分类 | 数量 | 状态 |
|----------|------|------|
| Blocking（必修复） | 0 | 已修复：create/getSchema/executors API |
| Optional（可优化） | 2 | 已记录 |

**自检结论：Passed**

### C6 开发后自检

| 检查项 | 状态 |
|--------|------|
| 组件复用 | ✅ |
| 命名规范 | ✅ |
| 页面结构统一 | ✅ |
| 状态完整（loading/empty/error） | ✅ |
| API规范 | ✅ |
| 权限兼容 | ✅ |
| 异常处理 | ✅ |
| 是否影响现网 | ✅ |

**自检结论：Pass**

## 6. 验收结果

| 验收项 | 状态 |
|--------|------|
| 页面是否能打开 | ✅ |
| 主流程是否可用 | ✅ |
| API是否成功调用 | ✅ |
| 是否存在报错 | ✅ |
| UI是否破坏 | ✅ |
| 是否影响旧功能 | ✅ |

**验收结论：通过**

## 7. 备注

- 执行器列表通过 `userApi.executors.list()` 获取，当前后端需支持用户端执行器查询
- PayloadEditor 支持 Schema 表单模式和 JSON 手动输入模式切换
- 表单提交后成功跳转至任务列表页
- 面包屑导航：首页 → 任务中心 → 新建任务
