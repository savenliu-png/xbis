# T008 ability API 层 — 设计评审报告

> 评审日期: 2026-04-24  
> 评审人: 技术负责人 + 架构审查官  
> 状态: Passed with changes

---

## 评审结果

### 状态: 👉 Passed with changes

---

## 问题列表

### 问题1: API 路径未区分用户端/管理端
**严重程度: 中**

当前所有 API 使用 `/api/v1/abilities`，但管理端能力管理需要独立的 admin API（如批量操作、状态切换）。

### 问题2: 缺少能力搜索/筛选 API
**严重程度: 中**

能力列表页需要支持关键词搜索、分类筛选、状态筛选，但 API 定义中未体现筛选参数。

### 问题3: 缺少能力订阅/取消订阅 API
**严重程度: 中**

能力订阅是核心业务流程，但 API 列表中缺少订阅相关接口。

---

## 修改建议

### 建议1: 区分用户端/管理端 API（必须修改）
```typescript
// 用户端 API
export const abilityApi = {
  list: (params?: AbilityListParams) => apiClient.get('/api/v1/abilities', { params }),
  get: (id: string) => apiClient.get(`/api/v1/abilities/${id}`),
  subscribe: (id: string) => apiClient.post(`/api/v1/abilities/${id}/subscribe`),
  unsubscribe: (id: string) => apiClient.post(`/api/v1/abilities/${id}/unsubscribe`),
};

// 管理端 API
export const adminAbilityApi = {
  list: (params?: AdminAbilityListParams) => apiClient.get('/admin-api/v1/abilities', { params }),
  update: (id: string, data: AbilityUpdateRequest) => apiClient.put(`/admin-api/v1/abilities/${id}`, data),
  delete: (id: string) => apiClient.delete(`/admin-api/v1/abilities/${id}`),
  publish: (id: string) => apiClient.post(`/admin-api/v1/abilities/${id}/publish`),
};
```

### 建议2: 补充筛选参数（必须修改）
```typescript
interface AbilityListParams {
  keyword?: string;
  category?: string;
  status?: string;
  page?: number;
  pageSize?: number;
}
```

### 建议3: 补充订阅 API（必须修改）
增加 subscribe/unsubscribe 接口。

---

## 是否允许进入开发

### 👉 YES（附条件通过）

**必须完成的条件**:
1. 区分用户端/管理端 API（建议1）
2. 补充筛选参数（建议2）
3. 补充订阅 API（建议3）

---

## 评审总结

| 检查项 | 结果 | 说明 |
|--------|------|------|
| 组件复用原则 | N/A | 无组件 |
| 页面模板使用 | N/A | 无页面 |
| 重复组件 | N/A | 无组件 |
| 分层规范 | ✅ | 符合 API 分层 |
| API 合理性 | ⚠️ | 未区分用户/管理端 |
| 状态设计完整性 | N/A | 无状态 |
| 能力平台架构 | ⚠️ | 缺少订阅 API |
| 过度设计 | ✅ | 不过度 |
| 未声明数据模型 | ⚠️ | 缺少筛选参数 |

**风险等级**: 中
