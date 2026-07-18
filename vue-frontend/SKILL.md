---
name: vue-frontend
description: >-
  Vue 3 前端开发规范（Agent-style UI）：Modern SaaS + Chat 布局，Slate + Blue 色板，
  纯 CSS 变量，无第三方 UI 库。涵盖技术栈、设计 token、布局、组件、Toast、
  composable、UI/UX 偏好与检查清单。
  Use when creating or editing Vue 3 + Pinia + Vite + TypeScript frontend,
  UI styling, pages/components, CSS, or when the user mentions UI 风格、
  界面优化、下拉框、侧边栏、按钮样式、前端规范。
---

# Vue Frontend — Agent-style UI

轻量 Modern SaaS + Chat 布局（侧栏 + 主内容，类 ChatGPT）。
色板审美接近 Tailwind Slate + Blue（项目未使用 Tailwind）。
**不是** Material / Ant Design / Element Plus；禁止引入这些库「换皮」。

详细 token / Method 色 / AppSelect 结构见 [tokens.md](tokens.md)。

---

## 1. 技术栈（固定）

| 项 | 选择 |
|----|------|
| 框架 | Vue 3 Composition API + Pinia + Vite + TypeScript |
| 样式 | 纯 CSS 变量：`styles/index.css`（token）+ `styles/app.css`（组件） |
| 字体 | `@fontsource/inter` → `--font: Inter` |
| 下拉 | 自制 `AppSelect.vue`，**禁止**原生 `<select>` |
| 图标 | 内联 SVG / Unicode / 文字；无统一 icon 库 |
| UI 库 | **禁止** Element Plus / Ant Design / 任何第三方 Design System |

改完 `web/` 任意文件 → **必须** `cd web && npm run build` 直到通过。

---

## 2. 设计 Token（摘要）

完整 `:root` 见 [tokens.md](tokens.md)。核心：

- 背景：`--bg` 白 / `--bg-soft #f8fafc` / `--bg-sidebar #f4f7fc`
- 选中：`--bg-sidebar-item-hover` / `--bg-sidebar-item-active`
- 主色：`--primary` / `--accent` `#2563eb`（禁止橙 / indigo）
- 形状：`--radius 12` / `--radius-sm 8` / `--radius-lg 16`
- 布局：`--page-gutter: clamp(16px, 2.5vw, 28px)` · `--content-max: 1200px`

---

## 3. 布局骨架

```
app-shell (grid: sidebar | main, 100vh, overflow hidden)
├── sidebar (#f4f7fc, 260px → 64px 折叠, transition 0.22s)
│   ├── BrandLogo (SVG mark + 文字)
│   ├── nav-list（nav-item = session-item 选中风格）
│   ├── AppSelect 全局上下文/环境选择器
│   └── UserBar
└── main (flex column, overflow hidden)
    └── page-view → page-shell
        ├── page-header（h1 宜小 + meta 灰色）
        └── split-pane / page-shell--scroll
```

**规则**：
- **无 TopBar**；全局选择器只在侧栏
- 分栏用 **flex + min-height:0**，**禁止** `height: calc(100vh - …)`
- 长页用 `.page-shell--scroll`
- 折叠侧栏 64px rail，图标按钮 40×40

### 典型信息架构（工具型后台）

| 视图 | 说明 |
|------|------|
| 资源目录 | 左树 + 右详情/调试器（Postman 式交互，非 DevTools 皮肤） |
| 环境/配置 | split-pane 列表 + 表单 |
| 多步用例/流程 | split-pane + 步骤编辑器 |
| 执行历史 / AI 编排 | 从属功能，样式同主规范 |

全局环境 → Send/运行默认带上；页面内勿重复造环境选择器（条目绑定 env 除外）。

---

## 4. 按钮 `.btn`

| 变体 | 样式 | 用途 |
|------|------|------|
| 默认 | 胶囊 `border-radius: 999px`，0.8125rem/500，轻阴影 | 通用 |
| `.btn-primary` | 蓝底白字 | 主操作 |
| `.btn-ghost` | 透明，hover → `--accent-soft` + accent | 次要 |
| `.btn-danger` | 白底，`#fecaca` 边框，hover → `#fef2f2` | 危险 |
| `.btn-sm` | 6×12 padding，0.75rem | 紧凑 |
| `.btn-icon` | 30×30 方形 | 图标按钮 |

交互：hover 色变 + active `scale(0.98)`（不用 `translateY`）；disabled `opacity: 0.45`。
文字按钮 `.btn-text`：最小 padding，用于表格/模态窗内联操作。

---

## 5. 表单控件

### 输入 `.input` / `.textarea`

- `--radius-sm`，8×12 padding
- Focus：primary 边框 + `box-shadow: 0 0 0 3px rgba(37,99,235,.12)`
- textarea：`--mono`，12.5px，min-height 120px；JSON body 用 `.textarea`

### 下拉 `AppSelect`

| variant | 用途 |
|---------|------|
| `default` | 表单、侧栏（胶囊 trigger + Teleport） |
| `bare` | URL/composer 内嵌（无边框） |
| `compact` | 表格内 |

- 菜单 **Teleport to body**，z-index 1000；chevron 选中旋转 180°
- 选项常量放 `select-options.ts`
- 结构见 [tokens.md](tokens.md)

### 键值对表格 `KVTable`

- grid：`1fr 1fr auto`，gap 8px
- 两个 `.input` + ghost 删除；底部 "+ 添加一行" ghost sm

---

## 6. 列表与选中态

**统一 session-item 风格**（禁止左侧蓝条 / `--primary-soft` 高亮）：

```css
.item:hover  { background: var(--bg-sidebar-item-hover); }
.item.active {
  background: var(--bg-sidebar-item-active);
  border-color: var(--border-sidebar-item-active);
}
```

适用：`.list-row`、`.nav-item`、任意可选项列表行。

- `.list-row`：竖排，10×12；`.name` 0.875rem/600，`.sub` 0.75rem muted；hover 显示删除
- 过滤 chip：水平 pill；active → primary 填充白字

### 搜索栏 `ListSearchBar`

- 胶囊容器，白底；focus-within → primary 边框 + focus ring
- 可内嵌可移除 filter tag；聚焦且无 tag 时弹出筛选建议（`mousedown.prevent`）

---

## 7. 模态窗

### 确认弹窗 `confirmDialog`

| 属性 | 值 |
|------|------|
| 遮罩 | `rgba(0,0,0,.35)` |
| 对话框 | 380px，`--radius-lg`，`padding-top: 18vh` |
| 按钮 | ghost 取消 + primary/danger 确认（靠右） |
| 动画 | scale 0.95→1，0.15s |

### 标准模态窗 `.modal-panel`

- 遮罩 `rgba(15,23,42,.45)`，`padding-top: 10vh`
- 面板 max 560px（宽版 864px `--wide`）
- **Header 必须** `background: var(--bg-soft)` + 底部分割线（禁止纯白 header）

---

## 8. Toast

- **位置**：fixed **右下**
- 种类：info / ok / error（对应色背景 + 边框）
- 动画：slide，0.2s；可选 action pill；自动消失约 3500ms
- **业务错误走 toast，禁止弹窗报错**

---

## 9. 标签、分栏、Composer、代码区

### 标签

- Method pill：GET 蓝 / POST 绿 / PUT 琥珀 / DELETE 红 等（色值见 tokens.md）
- Status pill：success 绿 / error 红 / running 蓝 / pending muted

### split-pane

```css
.split-pane {
  display: grid;
  grid-template-columns: minmax(300px, 360px) minmax(0, 1fr);
  gap: 14px;
  flex: 1;
  min-height: 0;
}
```

- `.list-panel` / `.detail-panel`：白底 + border + radius + shadow
- 列表 header 用 `--bg-soft`；详情 sticky header + scroll body

### Composer / URL 一体条

```css
.composer-row, .tester-url-row {
  border-radius: var(--radius-lg);
  border: 1.5px solid var(--border-strong);
}
```

- bare AppSelect（可选）+ mono/普通 input + 发送按钮；**亮色主题**

### 代码 / 响应面板

- 深色底 `#0f172a` + `#e2e8f0` + mono 12px
- JSON 高亮：key `#7c3aed`，string `#059669`，number `#d97706`，boolean `#2563eb`
- 可折叠（chevron）

### 空状态

`.empty-state` / `.lockout-screen`：居中、轻 typography；错误不弹窗。

---

## 10. 动画与排版

| 场景 | 时长 | 效果 |
|------|------|------|
| hover 色变 | 0.12–0.15s | background/color |
| Toast | 0.2s | opacity + translate |
| 确认弹窗 | 0.15s | fade + scale |
| 侧栏折叠 | 0.22s | grid-template-columns |
| 下拉 chevron | 0.15s | rotate 180° |
| 按钮 active | 0.1s | scale(0.98) |
| 加载旋转 | 0.7–1s linear | infinite |

| 用途 | 大小 | 字重 |
|------|------|------|
| 正文 | 0.875rem | 400 |
| 页面标题 h1 | **0.875rem** | 500（宜小，勿抢视觉） |
| 分区标题 | 0.9375rem | 600 |
| 列表名称 | 0.875rem | 600 |
| 辅助/meta | 0.75–0.8125rem | 400 |
| 微标签 | 0.6875rem | 500–600 |
| 代码 | 12–12.5px | 400 |

---

## 11. Composable 模式

```typescript
// 归属过滤（配合 ListSearchBar tag + suggest）
const { ownerFilter, searchFocused, ownerItems } = useOwnerFilter(items, isMine)

// 列表 + 详情草稿编辑
const { selectedID, draft, isNew, startNew, finishCreate } = useDraftEditor(items, toID)
```

选中时 deep clone 为 draft；新建时 `isNew=true`。

---

## 12. 全局约定

1. 错误 → toast；破坏操作 → `confirmDialog`（不用 `window.confirm`）
2. 模态窗 / 卡片 header → `--bg-soft`
3. 列表选中 = session-item 灰高亮（非 primary-soft）
4. 下拉统一 AppSelect（Teleport body）
5. 侧栏全局选择器生效，页面内不重复
6. flex + min-height:0，禁止 calc(100vh)
7. 滚动条：10px，thumb `#cfd2dc`，radius 5px
8. 卡片：白底 + `--border` + `--radius` + `--shadow`（勿 Material elevation）

### 关键文件（典型路径，按项目调整）

| 文件 | 职责 |
|------|------|
| `web/src/styles/index.css` | 全局 token |
| `web/src/styles/app.css` | 布局 + 组件样式 |
| `web/src/App.vue` | app-shell + page-view |
| `web/src/components/layout/Sidebar.vue` | 侧栏 + 环境 + UserBar |
| `web/src/components/common/AppSelect.vue` | 统一下拉 |
| `web/src/utils/select-options.ts` | 下拉选项常量 |

---

## 13. UI/UX 偏好（必须遵守）

### 色彩与质感
- 蓝色主色 `#2563eb`，**禁止橙色** / indigo
- 轻量质感：轻阴影、细边框、圆角胶囊
- 亮色主题；代码/响应 body 可深色 `#0f172a`
- placeholder `#cbd5e1`；标签彩色 pill，不要单色灰底

### 空间与密度
- 紧凑优先，后台内容区拉伸填满
- h1 要小（0.875rem）；模态 header 紧凑（title + hint 同行）
- 侧栏不用分割线，用文字标签分组；全局选择器与 logo 保持间距

### 信息层级
- 能去掉就去掉；概念要少
- 辅助信息默认折叠（`?` / chevron）
- 颜色已区分状态时可去掉文字状态列；下拉过长则截断

### 交互
- 不要空白面板 — 打开即预览第一项
- 空列表用显眼大按钮引导新建
- 删除二次确认；tab 关闭 ≠ 删除
- 破坏性操作先展示 diff；确认/取消靠右
- hover 显示次要操作；展开用旋转 chevron
- 图标语义准确（导入朝下、同步旋转箭头）
- 操作按钮可识别（加边框、缩小字体）

### 视觉符号
- **不要 emoji**；扁平 stroke SVG + currentColor
- sparkle 用四角星，不要五角星
- 品牌 logo：SVG mark + 文字，不要纯文字

### 布局偏好
- 无 TopBar；侧栏 stroke 18×18 简约 icon
- 核心功能排第一；首页进主工作流
- 多会话可用浏览器 tab（`+` / `×`，关闭 ≠ 删除）
- 发送按钮圆形在输入区右下；附件紧挨其左

### 反面案例（明确否定）
- Element Plus / Ant Design 等 UI 库
- DevTools 深色三栏皮肤
- 纯文字 logo、橙色配色、emoji icon
- 大段文字上滑淡入等多余过渡
- 全局深色底白字、列表项内堆叠过多标签

---

## 14. 禁止项

- ❌ 第三方 UI 库 / 原生 `<select>`
- ❌ TopBar
- ❌ 弹窗报业务错（用 toast）
- ❌ 偏离 token 的 accent（除非用户明确要求）
- ❌ Postman DevTools 深色三栏皮肤

---

## 15. 新组件检查清单

```
- [ ] 复用现有 token / class，不引入新 UI 库
- [ ] 列表选中 = session-item 风格
- [ ] 下拉用 AppSelect + select-options
- [ ] 布局 flex + min-height:0，无 calc(100vh)
- [ ] 错误走 toast；模态 header 用 --bg-soft
- [ ] 按钮胶囊形 + active scale(0.98)
- [ ] 不要 emoji，用扁平 SVG icon
- [ ] 默认预览第一项；辅助信息折叠；概念尽量少
- [ ] npm run build 通过
```
