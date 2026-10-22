# 个人计划管理平台 — 设计规范

> 版本：v1.0  
> 日期：2026-07-12  
> 状态：已确认，待实现

---

## 1. 项目概述

一款纯前端本地网页应用，用于个人日常计划管理、习惯追踪与 AI 辅助规划。  
**定位**：个人规划中枢，最终覆盖「每日待办 → 习惯养成 → 长期目标 → 学习管理」全流程。

**技术选型**：单 HTML 文件（内嵌 CSS + JS），零依赖、零构建，双击即用。  
**数据存储**：浏览器 IndexedDB，本地优先，支持 JSON 导入/导出备份。  
**AI 接入**：双通道——Hermes 本地桥接（v1）+ DeepSeek API 直连（v2 预留）。

---

## 2. 分阶段规划

| 阶段 | 模块 | 说明 |
|------|------|------|
| **v1** | 每日待办 + 习惯打卡 + 数据看板 + AI（Hermes） | 核心底座，先跑起来 |
| v2 | 长期目标追踪 | 目标 → 子任务拆解、进度追踪 |
| v3 | 学习/阅读管理 + Obsidian 联动 | 书单、课程进度、笔记关联 |

---

## 3. 架构设计

### 3.1 文件结构

```
D:/projects/003/
├── index.html              ← 唯一源文件（HTML + CSS + JS）
├── docs/
│   └── specs/
│       └── 2026-07-12-personal-planner-design.md   ← 本文档
└── README.md
```

### 3.2 代码模块（JS 类）

| 类 | 职责 | 对外接口 |
|----|------|----------|
| `Database` | IndexedDB 封装，管理 tasks/habits/settings 三张表 | `getAll()`, `add()`, `update()`, `delete()` |
| `TaskManager` | 任务 CRUD + 排序 + 子任务展开折叠 | `addTask()`, `toggleComplete()`, `reorder()`, `addSubtask()` |
| `HabitManager` | 习惯打卡 + 连续天数计算 | `checkin()`, `getStreak()`, `getHeatmap()` |
| `Dashboard` | 看板：统计卡片、折线图、热力图、环形图 | `render()` |
| `AIChatProvider` | AI 接口抽象（后续 HermesProvider / DeepSeekProvider 实现） | `send(messages, context)` |
| `HermesProvider` | v1 实现：通过本地 HTTP/WebSocket 与 Hermes 通信 | 继承 AIChatProvider |
| `Settings` | 主题切换、数据导入导出、API Key 管理 | `setTheme()`, `exportData()`, `importData()` |
| `Router` | 侧边栏导航 → 视图切换 | `navigate(view)` |
| `App` | 顶层应用入口，初始化各模块 | `init()` |

### 3.3 数据模型

**tasks 表**
```json
{
  "id": "uuid",
  "title": "完成项目报告",
  "notes": "需要包含数据分析和总结",
  "completed": false,
  "priority": "high",
  "tags": ["工作"],
  "dueDate": "2026-07-12",
  "parentId": null,
  "order": 1,
  "createdAt": "2026-07-12T08:00:00Z",
  "completedAt": null
}
```

**habits 表**
```json
{
  "id": "uuid",
  "title": "晨间阅读",
  "description": "每天阅读至少30分钟",
  "frequency": "daily",
  "color": "#4f46e5",
  "currentStreak": 7,
  "bestStreak": 15,
  "checkins": { "2026-07-12": true, "2026-07-11": true, ... },
  "createdAt": "2026-07-01T00:00:00Z"
}
```

**settings 表**
```json
{
  "key": "theme",
  "value": "light"
}
```

---

## 4. 页面布局

```
┌──────────┬──────────────────────────────────┐
│  侧边栏   │       主内容区                     │
│  (220px) │                                   │
│          │  顶部栏：当前视图标题 + 日期         │
│ 📊 总览   │  ──────────────────────────────── │
│ 📅 今日   │                                   │
│ ✅ 待办   │  内容区（随导航切换）               │
│ 🔥 习惯   │                                   │
│ 🎯 目标   │                                   │
│ 📚 学习   │                                   │
│ ──────── │                                   │
│ ⚙️ 设置   │                                   │
│          │  ┌─── AI 助手（右下角悬浮）──────┐  │
│          │  │  💬                          │  │
│          │  └──────────────────────────────┘  │
└──────────┴──────────────────────────────────┘
```

### 4.1 各视图说明

#### 📊 总览（看板）
- 4 张统计卡片（本周完成、本月完成、总任务、连续打卡天数）
- 本周完成趋势折线图（Canvas）
- 习惯热力图（最近 4 周，GitHub 贡献图风格）
- 任务分类环形图（按标签分布）
- AI 周报入口按钮

#### 📅 今日
- 今日日期 + 星期（大标题）
- 待完成任务列表（可勾选、展开子任务）
- 今日习惯打卡行（每行一个习惯，点击打卡）
- 快速添加任务输入框

#### ✅ 待办
- 全部任务列表（可按标签/优先级筛选）
- 拖拽排序
- 展开/折叠子任务
- 批量操作（全选、删除选中）

#### 🔥 习惯
- 习惯列表（每个习惯：名称 + 今日状态 + 连续天数）
- 添加/编辑习惯
- 习惯详情弹窗（月度热力图 + 统计数据）

#### ⚙️ 设置
- 主题切换（浅色 / 深色 / 跟随系统）
- AI 配置（Hermes 连接状态、API Key 输入框）
- 数据管理（导出 JSON、导入 JSON、清空所有数据）
- 关于信息

---

## 5. AI 桥接设计

### 5.1 接口抽象

```javascript
class AIChatProvider {
  async send(messages, context) { throw new Error('Not implemented'); }
  async stream(messages, context) { throw new Error('Not implemented'); }
}
```

- `messages`：对话历史 `[{role, content}]`
- `context`：当前页面数据快照 `{tasks, habits, view}`

### 5.2 Hermes 通道（v1）

通过本地 HTTP POST 与 Hermes 本地服务通信。

工作流：
1. 用户在 AI 面板输入自然语言
2. 系统打包当前数据 + 对话历史 → POST 到 Hermes
3. Hermes 返回结构化计划（JSON）
4. 前端渲染为可交互的任务卡片
5. 用户一键采纳或手动调整

### 5.3 DeepSeek 通道（v2 预留）

实现 `AIChatProvider` 接口，通过 fetch 直连 DeepSeek API。

### 5.4 AI 能力清单

| 场景 | 用户输入示例 | AI 输出 |
|------|-------------|---------|
| 目标拆解 | "30天学完 Python 基础" | 按周拆分的任务列表 + 每日时间建议 |
| 每日规划 | "今天：写代码、学英语、健身" | 排好优先级和时间段的计划 |
| 周报总结 | "总结本周" | 基于实际打卡数据的自然语言总结 |
| 习惯分析 | "分析我的习惯" | 模式识别 + 优化建议 |

---

## 6. 视觉设计规范

### 6.1 设计语言

- **风格**：简洁现代 + 高级质感（Linear / Notion 方向）
- **圆角**：卡片 12px，按钮 8px，输入框 8px
- **阴影**：卡片 `0 1px 3px rgba(0,0,0,0.08)`，悬浮 `0 4px 12px rgba(0,0,0,0.12)`
- **间距**：8px 基准网格（8/16/24/32/48）
- **动效**：`transition: 150ms ease`，hover 微浮、完成划线动画

### 6.2 配色

**浅色主题**
| 用途 | 色值 |
|------|------|
| 背景 | `#f8f9fa` |
| 卡片 | `#ffffff` |
| 主文字 | `#1a1a2e` |
| 次要文字 | `#6b7280` |
| 强调色 | `#4f46e5`（靛蓝） |
| 成功 | `#10b981` |
| 高优先级 | `#ef4444` |

**深色主题**
| 用途 | 色值 |
|------|------|
| 背景 | `#0f0f23` |
| 卡片 | `#1a1a2e` |
| 主文字 | `#e5e7eb` |
| 强调色 | `#818cf8` |

### 6.3 字体

- 中文：系统默认（微软雅黑 / PingFang SC）
- 英文/数字：`Inter` 或系统等宽字体
- 数字使用 `font-variant-numeric: tabular-nums` 防止跳动

### 6.4 图标

- 全局使用 emoji + 纯 CSS 绘制
- 不引入任何图标库，保持零依赖

---

## 7. 非功能需求

- **性能**：页面加载 < 500ms，AI 响应 < 10s
- **数据安全**：所有数据存本地，不上传任何服务器（AI 调用除外）
- **兼容性**：Chrome / Edge / Firefox 最新版
- **代码质量**：每个 JS 类职责单一，对外接口清晰，便于后续扩展

---

## 8. 验收标准

- [ ] 双击 index.html 即可在浏览器中正常运行
- [ ] 可以添加、编辑、删除、勾选完成任务
- [ ] 可以创建习惯并每日打卡，显示连续天数
- [ ] 看板页面展示统计数据和图表
- [ ] AI 面板可通过 Hermes 生成计划
- [ ] 深色/浅色主题切换正常
- [ ] 数据导出 JSON 后，清空再导入，数据完整恢复
- [ ] 所有交互有合理的过渡动画，无明显卡顿

---

*文档版本：v1.0 · 最后更新：2026-07-12*
