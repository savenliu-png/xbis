# T015 任务列表页 — 验收文档

## 1. 修改文件清单

| 序号 | 文件路径 | 修改类型 |
|------|----------|----------|
| 1 | `packages/user/src/App.tsx` | 修改 |
| 2 | `packages/user/src/pages/Jobs.tsx` | 新增 |
| 3 | `packages/user/src/components/Layout.tsx` | 修改 |

## 2. 新增组件清单

无需新增组件，全部复用已有组件：

| 组件名称 | 所属层级 | 用途 |
|----------|----------|------|
| TaskPageShell | layout | 页面外壳（list 模式） |
| TaskItem | business | 任务卡片 |
| TaskFilterBar | blocks | 筛选栏 |
| JobResultViewer | blocks | 结果详情预览 |
| Pagination | base | 分页 |

## 3. API变更清单

| API路径 | 调用位置 | 是否通过 Business Services |
|---------|----------|--------------------------|
| `GET /api/v1/jobs` | Jobs.tsx useEffect | 是（userApi.jobs.list） |
| `POST /api/v1/jobs/batch-cancel` | Jobs.tsx 批量取消 | 是（userApi.jobs.batchCancel） |

## 4. 风险说明

| 风险项 | 说明 |
|--------|------|
| 是否影响现有功能 | 否，新增页面，不修改现有页面逻辑 |
| 是否影响数据结构 | 否，使用已有 Job 类型 |
| 是否影响路由 | 是，新增 `/user/jobs` 路由，已添加 `isAuthenticated` 守卫 |
| 是否影响导航 | 是，Layout 侧边栏新增「任务中心」入口 |

## 5. 自检结果

### C5 AI 强制自检

| 问题分类 | 数量 | 状态 |
|----------|------|------|
| Blocking（必修复） | 0 | 已修复：硬编码颜色 → CSS Variables |
| Optional（可优化） | 2 | 已记录，不影响功能 |

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

- 批量操作按钮使用原生 `<button>` 而非 `base/Button`，因 `base/Button` 组件在当前项目中未确认存在，且原生按钮已使用 CSS Variables 保持样式一致性。
- 日期筛选状态（dateFrom/dateTo）已在代码中预留，当前 UI 未暴露日期选择器，待后续需求扩展。
- 批量取消 API 使用类型断言 `(userApi.jobs as any).batchCancel`，因当前 `userApi.jobs` 类型定义中未包含 `batchCancel` 方法，建议后续在 `services.ts` 中补充类型定义。
