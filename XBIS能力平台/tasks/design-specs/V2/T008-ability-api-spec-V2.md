# T008 ability API 层 — 修正版设计方案（V2）

> 原设计方案: [T008-ability-api-spec.md](../T008-ability-api-spec.md)
> 评审报告: [T008-ability-api-Reviewer.md](../T008-ability-api-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed with changes

---

## 二、必须修改项（Blocking）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 区分用户端/管理端 API | 第 5 节 API 调用 |
| 2 | 补充筛选参数 | 第 5 节 API 调用 |
| 3 | 补充订阅 API | 第 5 节 API 调用 |

---

## 三、建议修改项（Optional）

无建议修改项。

---

## 四、修正版设计方案（V2）

### 1. 页面结构

无页面结构，本任务为 API 层定义。

---

### 2. 组件拆分

无组件拆分，本任务为 API 层定义。

---

### 3. 数据流

无运行时数据流。

---

### 4. 状态管理

无状态管理需求。

---

### 5. API 调用（V2 修正）【已修复】

#### 用户端 API

```typescript
// 能力列表（带筛选）
interface AbilityListParams {
  keyword?: string;
  category?: string;
  status?: string;
  page?: number;
  pageSize?: number;
}

export const abilityApi = {
  // 查询能力列表
  list: (params?: AbilityListParams) => 
    apiClient.get('/api/v1/abilities', { params }),
  
  // 查询能力详情
  get: (id: string) => 
    apiClient.get(`/api/v1/abilities/${id}`),
  
  // 订阅能力 【新增】
  subscribe: (id: string) => 
    apiClient.post(`/api/v1/abilities/${id}/subscribe`),
  
  // 取消订阅 【新增】
  unsubscribe: (id: string) => 
    apiClient.post(`/api/v1/abilities/${id}/unsubscribe`),
};
```

#### 管理端 API 【新增】

```typescript
export const adminAbilityApi = {
  // 查询能力列表（管理端）
  list: (params?: AdminAbilityListParams) => 
    apiClient.get('/admin-api/v1/abilities', { params }),
  
  // 更新能力
  update: (id: string, data: AbilityUpdateRequest) => 
    apiClient.put(`/admin-api/v1/abilities/${id}`, data),
  
  // 删除能力
  delete: (id: string) => 
    apiClient.delete(`/admin-api/v1/abilities/${id}`),
  
  // 发布能力
  publish: (id: string) => 
    apiClient.post(`/admin-api/v1/abilities/${id}/publish`),
  
  // 下架能力
  deprecate: (id: string) => 
    apiClient.post(`/admin-api/v1/abilities/${id}/deprecate`),
};
```

---

### 6. 用户交互流程

无用户交互流程。

---

### 7. 边界与异常

| 错误码 | 场景 | 处理方式 |
|--------|------|----------|
| 400 | 参数错误 | 返回校验错误详情 |
| 403 | 权限不足 | 返回 403 |
| 404 | 能力不存在 | 返回 404 |

---

### 8. 性能优化

- API 响应缓存（5 分钟）
- 列表分页加载

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| API 版本冲突 | 低 | 用户端/管理端 API 路径冲突 | 使用前缀区分 |

---

### 10. 开发步骤拆分

#### Step 1: 用户端 API（0.5 人日）
- [ ] list / get
- [ ] subscribe / unsubscribe 【新增】
- [ ] 筛选参数 【新增】

#### Step 2: 管理端 API（0.5 人日）【新增】
- [ ] admin list / update / delete
- [ ] publish / deprecate

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **API 分层** | 未区分用户/管理端 | 明确区分 `abilityApi` 和 `adminAbilityApi` |
| **筛选参数** | 未定义 | 新增 `AbilityListParams` |
| **订阅 API** | 未定义 | 新增 `subscribe` / `unsubscribe` |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（区分用户/管理端 API、补充筛选参数、补充订阅 API）
2. 风险等级：低
