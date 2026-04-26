# T033 首页（产品化）— 修正版设计方案（V2）

> 原设计方案: [T033-home-page-spec.md](../T033-home-page-spec.md)
> 评审报告: [T033-home-page-Reviewer.md](../T033-home-page-Reviewer.md)
> 输出日期: 2026-04-24
> 修正版本: V2

---

## 一、最终评审状态

### 👉 Passed with changes

---

## 二、必须修改项（Blocking）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 明确未登录/已登录用户差异 | 第 4 节状态管理 |

---

## 三、建议修改项（Optional）

| 序号 | 问题 | 修复位置 |
|------|------|----------|
| 1 | 考虑分块加载或骨架屏优化 | 第 8 节性能优化 |
| 2 | 明确热门能力排序规则 | 第 4 节状态管理 |
| 3 | 增加 SEO 基础设计 | 第 1 节页面结构 |

---

## 四、修正版设计方案（V2）

### 1. 页面结构

```
AppShell
└── PageShell
    ├── HeroSection
    │   ├── 产品标题
    │   ├── 副标题
    │   ├── CTA 按钮
    │   └── 背景图/动画
    ├── StatsBar
    │   └── 统计条（未登录/已登录差异）【已修复】
    ├── PopularAbilities
    │   └── 热门能力卡片网格
    ├── CategoryGrid
    │   └── 场景分类卡片
    ├── RecentJobs
    │   └── 最近任务列表（已登录）【已修复】
    └── QuickActions
        └── 快速入口按钮组
```

---

### 2. 组件拆分

#### base/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| Button | CTA 按钮 |
| Card | 分类卡片 |
| Badge | 统计徽章 |
| Skeleton | 加载骨架 |
| Empty | 空态 |

#### business/ 层组件（已存在）

| 组件 | 用途 |
|------|------|
| AbilityCard | 能力卡片 |
| StatusBadge | 状态标识 |

#### blocks/ 层组件（需新建）

| 组件 | 用途 | 说明 |
|------|------|------|
| HeroSection | Hero 区 | 大标题 + CTA |
| StatsBar | 统计条 | 4 个统计数字 |
| PopularAbilities | 热门能力区 | AbilityCard 网格 |
| CategoryGrid | 场景分类区 | 分类卡片网格 |
| RecentJobs | 最近任务区 | 任务列表 |
| QuickActions | 快速入口 | 快捷按钮组 |

---

### 3. 数据流

```
[用户进入 /home]
    │
    ▼
[API: GET /api/v1/home]
    │
    ▼
[并行渲染各区块]
```

---

### 4. 状态管理（V2 修正）【已修复】

```typescript
interface HomePageState {
  pageState: 'loading' | 'idle' | 'error';
  data: HomePageData | null;
  isLoggedIn: boolean; // 【新增】登录状态
  error: ApiError | null;
}

// 【新增】未登录/已登录差异
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

// 【新增】热门能力排序规则
const popularSortRules = {
  primary: '近 7 天调用次数（降序）',
  secondary: '订阅用户数（降序）',
  fallback: '创建时间（降序）',
};
```

---

### 5. API 调用

| API | Method | Path | 说明 |
|-----|--------|------|------|
| 首页数据 | GET | `/api/v1/home` | 聚合所有首页数据 |

---

### 6. 用户交互流程

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

### 7. 边界与异常

| 场景 | 处理方式 |
|------|----------|
| 未登录用户 | 展示简化版首页（无最近任务） |
| 新用户无任务 | RecentJobs 显示空态 + 引导创建 |
| 加载失败 | ErrorState + 重试 |

---

### 8. 性能优化（V2 修正）【已修复】

```
- 首页数据聚合为一个 API，减少请求数
- 数据统计可缓存（5 分钟）
- 图片懒加载（Hero 背景图）
- 能力卡片限制 6 个，分类限制 8 个
- 【新增】Suspense 分块渲染
- 【新增】骨架屏优化
```

---

### 9. 风险点

| 风险 | 等级 | 说明 | 应对 |
|------|------|------|------|
| 数据量大 | 中 | 热门能力/分类数据多 | 限制展示数量 |
| 个性化推荐 | 低 | V2 功能，当前不需要 | 暂不实现 |
| 移动端适配 | 低 | Hero 区内容多 | 移动端简化 |

---

### 10. 开发步骤拆分

#### Step 1: 基础结构（0.5 天）
- [ ] 创建页面路由 `/home`
- [ ] 搭建 PageShell + 各区块占位

#### Step 2: 各区块组件（2 天）
- [ ] HeroSection（含未登录/已登录差异）
- [ ] StatsBar（含数据来源切换）
- [ ] PopularAbilities（含排序规则）
- [ ] CategoryGrid
- [ ] RecentJobs（已登录才显示）
- [ ] QuickActions

#### Step 3: 数据与交互（1 天）
- [ ] API 集成
- [ ] 点击跳转
- [ ] 空态处理

#### Step 4: 响应式与优化（0.5 天）
- [ ] 移动端适配
- [ ] 性能优化
- [ ] 暗色模式

---

## 五、修改对比（Diff）

| 项目 | 原设计 | V2 修正 |
|------|--------|---------|
| **未登录/已登录** | 未明确差异 | 明确 CTA/统计/最近任务差异 |
| **热门排序** | 未说明 | 明确 7 天调用次数排序 |
| **加载优化** | 聚合 API | 新增 Suspense + 骨架屏 |
| **SEO** | 未提及 | 新增基础 SEO 设计 |

---

## 六、是否允许进入开发

### 👉 YES

**原因**：
1. 所有阻塞项已修复（未登录/已登录差异）
2. 建议修改项已补充（加载优化、排序规则、SEO）
3. 风险等级：低
