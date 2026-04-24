# 首页（产品化）

## 1. 基本信息

| 字段 | 内容 |
|------|------|
| 功能名称 ⭐ | 首页（产品化） |
| 任务编号 | T033 |
| 所属模块 ⭐ | M4 能力平台 |
| 优先级 | P1 |
| 指派给 | @frontend-dev |
| 预计工期 | 4 人日 |
| 创建日期 | 2026-04-23 |
| 目标上线日期 | 2026-05-18 |

## 2. 需求背景

### 2.1 问题描述
当前项目缺乏产品化首页，需要：
- Hero 区展示
- 热门能力推荐
- 场景分类
- 最近任务

### 2.2 目标用户
- 终端用户：了解产品
- 新用户：快速入门

### 2.3 预期效果
- 提供产品化首页
- 支持能力发现
- 支持快速入口

## 3. 页面位置 ⭐

### 3.1 用户端
| 页面 | 路由 | 入口 |
|------|------|------|
| 首页 | `/home` | 主导航「首页」 |

### 3.2 管理端（如适用）
无

### 3.3 页面原型/设计稿
- 参考：`packages/pages/home/HomePage`

## 4. 功能范围

### 4.1 包含功能
- [ ] Hero 区（产品标语 + CTA）
- [ ] 热门能力推荐
- [ ] 场景分类
- [ ] 最近任务
- [ ] 快速入口
- [ ] 数据统计展示
- [ ] 空态/加载态/错误态

### 4.2 不包含功能（明确排除）
- 个性化推荐（V2 迭代）
- 动态内容（V2 迭代）
- 社区内容（V2 迭代）

## 5. 数据结构 ⭐

### 5.1 新增/修改的数据模型

```typescript
// 首页数据
export interface HomePageData {
  hero: HeroData;
  popularAbilities: AbilityCardItem[];
  categories: CategoryItem[];
  recentJobs: RecentJobItem[];
  stats: HomeStats;
}

// Hero 数据
export interface HeroData {
  title: string;
  subtitle: string;
  ctaText: string;
  ctaLink: string;
  backgroundImage?: string;
}

// 最近任务项
export interface RecentJobItem {
  id: string;
  jobId: string;
  abilityName: string;
  status: JobStatus;
  createdAt: string;
}

// 首页统计
export interface HomeStats {
  totalAbilities: number;
  totalJobs: number;
  totalUsers: number;
  successRate: number;
}
```

### 5.2 数据库变更（如适用）
无

### 5.3 类型定义位置
- 文件：`packages/shared/src/types/index.ts`
- 命名：`HomePageData`, `HeroData`, `RecentJobItem`, `HomeStats`

## 6. 交互流程

### 6.1 主流程
```
[用户进入首页] ──► [加载首页数据] ──► [展示内容] ──► [用户浏览/点击]
```

### 6.2 异常流程
| 场景 | 触发条件 | 系统行为 | 用户反馈 |
|------|----------|----------|----------|
| 加载失败 | 网络错误 | 显示错误态 | 提示刷新 |
| 无数据 | 新用户无任务 | 显示空态 | 提示创建任务 |

### 6.3 边界情况
- 大量能力：限制展示数量
- 暗色模式：自动适配

## 7. 依赖组件 ⭐

### 7.1 基础组件（base/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| Button | 操作按钮 | 是（T002） |
| Card | 卡片 | 是（T002） |
| Badge | 徽章 | 是（T002） |
| Skeleton | 加载骨架 | 是（T002） |
| Empty | 空态展示 | 是（T002） |

### 7.2 业务组件（business/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| AbilityCard | 能力卡片 | 是（T003） |
| StatusBadge | 状态标识 | 是（T003） |

### 7.3 页面块组件（blocks/）
| 组件 | 用途 | 是否已存在 |
|------|------|-----------|
| HeroSection | Hero 区 | 否 |
| PopularAbilities | 热门能力区 | 否 |
| CategoryGrid | 场景分类区 | 否 |
| RecentJobs | 最近任务区 | 否 |
| StatsBar | 统计条 | 否 |

### 7.4 页面模板（layout/）
| 模板 | 用途 |
|------|------|
| PageShell | 页面骨架 |

### 7.5 需新建组件
| 组件 | 所属分层 | 说明 |
|------|----------|------|
| HeroSection | blocks | Hero 区 |
| PopularAbilities | blocks | 热门能力区 |
| CategoryGrid | blocks | 场景分类区 |
| RecentJobs | blocks | 最近任务区 |
| StatsBar | blocks | 统计条 |

## 8. API 接口 ⭐

### 8.1 新增接口

#### 首页数据查询
- **Method**: GET
- **Path**: `/api/v1/home`
- **响应类型**: `HomePageData`
- **权限**: 用户（已登录）

**响应示例**:
```json
{
  "success": true,
  "data": {
    "hero": {
      "title": "AI 能力平台",
      "subtitle": "发现、测试、接入 AI 能力",
      "ctaText": "开始探索",
      "ctaLink": "/abilities"
    },
    "popularAbilities": [...],
    "categories": [
      { "key": "ai", "label": "AI 模型", "count": 24 },
      { "key": "nlp", "label": "自然语言", "count": 18 }
    ],
    "recentJobs": [
      {
        "id": "job-001",
        "jobId": "job-20260424-001",
        "abilityName": "文本生成",
        "status": "completed",
        "createdAt": "2026-04-24T10:00:00Z"
      }
    ],
    "stats": {
      "totalAbilities": 128,
      "totalJobs": 125800,
      "totalUsers": 5680,
      "successRate": 98.5
    }
  }
}
```

### 8.2 修改接口
无

### 8.3 接口定义位置
- 文件：`packages/shared/src/api/services.ts`
- 命名：`userApi.home`

## 9. 验收标准 ⭐

### 9.1 功能验收
- [ ] Hero 区展示正常
- [ ] 热门能力推荐正常
- [ ] 场景分类展示正常
- [ ] 最近任务展示正常
- [ ] 快速入口可点击
- [ ] 数据统计展示正常
- [ ] 空态/加载态/错误态完整

### 9.2 技术验收
- [ ] TypeScript 类型完整，无 `any`
- [ ] 使用 Design Tokens，无散落样式
- [ ] 使用页面模板骨架
- [ ] 页面状态机完整（loading/empty/error/idle）
- [ ] 组件复用符合分层规范
- [ ] API 错误处理完整

### 9.3 性能验收
- [ ] 首屏加载 < 2s
- [ ] 无内存泄漏（useEffect 清理）

### 9.4 兼容性验收
- [ ] 桌面端 ≥1200px 正常显示
- [ ] 用户端移动端核心功能可用

## 10. 风险评估

### 10.1 是否影响现网 ⭐
- [x] **否** — 全新功能，不影响现有页面

### 10.2 是否依赖其他任务先完成 ⭐
- [x] **是** — 依赖任务：
  - [x] `T011` — 能力中心列表页 — 待开发
  - [x] `T015` — 任务列表页 — 待开发

### 10.3 技术风险
| 风险 | 可能性 | 影响 | 应对措施 |
|------|--------|------|----------|
| 数据量大 | 中 | 中 | 限制展示数量 |
| 个性化推荐 | 低 | 低 | 暂不实现 |
| 移动端适配 | 中 | 低 | 响应式布局 |

## 11. 开发检查清单

### 11.1 开发前
- [ ] 需求已评审并确认
- [ ] 设计稿已确认
- [ ] 数据结构和 API 已评审
- [ ] 依赖任务已完成

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

- 首页为产品化入口，需突出价值主张
- 支持暗色模式
- 数据统计可缓存

---

**任务状态**：待评审 → 评审通过 → 开发中 → 自测完成 → CR 中 → 联调中 → 已上线
