# T033 首页（产品化）- 最终设计方案（Final）

## 1. 页面结构

### 1.1 层级结构
```
AppShell
└── PageShell
    ├── HeroSection
    │   ├── 产品标题
    │   ├── 副标题
    │   ├── CTA 按钮（未登录/已登录差异）
    │   └── 背景图/动画
    ├── StatsBar
    │   └── 统计条（未登录展示平台数据/已登录展示个人数据）
    ├── PopularAbilities
    │   └── 热门能力卡片网格（4-6 个）
    ├── CategoryGrid
    │   └── 场景分类卡片（2 行 x 4 列）
    ├── RecentJobs（已登录才显示）
    │   └── 最近任务列表（5 条）
    └── QuickActions
        └── 快速入口按钮组
```

### 1.2 模板使用
| 区域 | 模板/组件 | 说明 |
|------|----------|------|
| 外层 | PageShell | 标准页面骨架 |
| 内容 | 自定义区块 | 多个 blocks 组合 |

### 1.3 布局说明
- **Desktop (≥1200px)**: 全宽布局，内容区 max-width 1200px 居中
- **Tablet (768-1199px)**: 网格调整为 2 列
- **Mobile (< 768px)**: 单列布局，Hero 区简化

---

## 2. 组件拆分

### 2.1 base/ 层组件（已存在）
| 组件 | 用途 |
|------|------|
| Button | CTA 按钮 |
| Card | 分类卡片 |
| Badge | 统计徽章 |
| Skeleton | 加载骨架 |
| Empty | 空态 |

### 2.2 business/ 层组件（已存在）
| 组件 | 用途 | 说明 |
|------|------|------|
| AbilityCard | 能力卡片 | T003 已开发 |
| StatusBadge | 状态标识 | T003 已开发 |

### 2.3 blocks/ 层组件（需新建）
| 组件 | 用途 | 说明 |
|------|------|------|
| HeroSection | Hero 区 | 大标题 + CTA |
| StatsBar | 统计条 | 4 个统计数字 |
| PopularAbilities | 热门能力区 | AbilityCard 网格 |
| CategoryGrid | 场景分类区 | 分类卡片网格 |
| RecentJobs | 最近任务区 | 任务列表 |
| QuickActions | 快速入口 | 快捷按钮组 |

---

## 3. 数据流

### 3.1 页面加载
```
[用户进入 /home]
    │
    ▼
[API: GET /api/v1/home]
    │
    ▼
[并行渲染各区块]
    ├── HeroSection
    ├── StatsBar
    ├── PopularAbilities
    ├── CategoryGrid
    ├── RecentJobs（已登录）
    └── QuickActions
```

### 3.2 数据流图
```
┌─────────────┐     ┌─────────────┐     ┌─────────────────────────────┐
│   Router    │────▶│  API GET    │────▶│        HomePageData          │
│   /home     │     │  /api/v1/home│     │  ┌─────────┐ ┌───────────┐ │
└─────────────┘     └─────────────┘     │  │  hero   │ │  stats    │ │
                                        │  └─────────┘ └───────────┘ │
                                        │  ┌─────────┐ ┌───────────┐ │
                                        │  │popular  │ │ categories│ │
                                        │  └─────────┘ └───────────┘ │
                                        │  ┌─────────┐               │
                                        │  │recentJobs│              │
                                        │  └─────────┘               │
                                        └─────────────────────────────┘
```

---

## 4. 状态管理

```typescript
interface HomePageState {
  pageState: 'loading' | 'idle' | 'error';
  data: HomePageData | null;
  isLoggedIn: boolean;
  error: ApiError | null;
}

// 未登录/已登录差异
const homePageVariants = {
  guest: {
    heroCta: '免费试用',
    statsSource: 'platform', // 平台总数据
    showRecentJobs: false,
  },
  authenticated: {
    heroCta: '创建任务',
    statsSource: 'personal', // 个人数据
    showRecentJobs: true,
  }
};

// 热门能力排序规则
const popularSortRules = {
  primary: '近 7 天调用次数（降序）',
  secondary: '订阅用户数（降序）',
  fallback: '创建时间（降序）',
};
```

---

## 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 首页数据 | GET | `/api/v1/home` | 聚合所有首页数据 |

### 响应结构
```typescript
interface HomePageData {
  hero: HeroData;
  popularAbilities: AbilityCardItem[];
  categories: CategoryItem[];
  recentJobs: RecentJobItem[];
  stats: HomeStats;
}
```

---

## 6. 用户交互流程

```
[进入首页]
    │
    ▼
[浏览 Hero 区]
    │
    ├── 未登录 ──► 点击 CTA「免费试用」──► 跳转注册
    ├── 已登录 ──► 点击 CTA「创建任务」──► 跳转任务创建
    │
    ▼
[浏览统计条]
    │
    ├── 未登录 ──► 展示平台总数据
    └── 已登录 ──► 展示个人数据
    │
    ▼
[浏览热门能力]
    │
    ├── 点击能力卡片 ──► 跳转能力详情
    │
    ▼
[浏览场景分类]
    │
    ├── 点击分类 ──► 跳转分类筛选后的能力列表
    │
    ▼
[浏览最近任务（已登录）]
    │
    ├── 点击任务 ──► 跳转任务详情
    │
    ▼
[点击快速入口]
```

---

## 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 未登录用户 | 展示简化版首页（无最近任务） |
| 新用户无任务 | RecentJobs 显示空态 + 引导创建 |
| 加载失败 | ErrorState + 重试 |

---

## 8. 性能优化

- 首页数据聚合为一个 API，减少请求数
- 数据统计可缓存（5 分钟）
- 图片懒加载（Hero 背景图）
- 能力卡片限制 6 个，分类限制 8 个
- Suspense 分块渲染
- 骨架屏优化

---

## 9. 风险点

| 风险 | 说明 | 应对方案 |
|------|------|----------|
| 数据量大 | 热门能力/分类数据多 | 限制展示数量；后端聚合 |
| 个性化推荐 | V2 功能，当前不需要 | 暂不实现 |
| 移动端适配 | Hero 区内容多 | 移动端简化 Hero，隐藏部分统计 |

---

## 10. 开发步骤

### Step 1: 基础结构（0.5 天）
- [ ] 创建页面路由 `/home`
- [ ] 搭建 PageShell + 各区块占位

### Step 2: 各区块组件（2 天）
- [ ] HeroSection（含未登录/已登录差异）
- [ ] StatsBar（含数据来源切换）
- [ ] PopularAbilities（含排序规则）
- [ ] CategoryGrid
- [ ] RecentJobs（已登录才显示）
- [ ] QuickActions

### Step 3: 数据与交互（1 天）
- [ ] API 集成
- [ ] 点击跳转
- [ ] 空态处理

### Step 4: 响应式与优化（0.5 天）
- [ ] 移动端适配
- [ ] 性能优化
- [ ] 暗色模式

---

## 附录：修改文件清单

### 新增文件
```
packages/pages/home/
├── index.tsx
├── HomePage.tsx
└── components/
    ├── HeroSection.tsx
    ├── StatsBar.tsx
    ├── PopularAbilities.tsx
    ├── CategoryGrid.tsx
    ├── RecentJobs.tsx
    └── QuickActions.tsx
```

### 修改文件
```
packages/shared/src/types/index.ts
packages/shared/src/api/services.ts
packages/router/index.tsx
```
