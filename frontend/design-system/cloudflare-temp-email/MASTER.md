# 设计系统（主文件）

> **逻辑说明：**实现具体页面前，优先查看 `design-system/cloudflare-temp-email/pages/[page-name].md`。
> 若页面文件存在，则页面规则 **覆盖** 本主文件；否则以本文件为准。

---

**项目：**Cloudflare Temp Email（临时邮箱）  
**生成：**2026-01-28  
**定位：**开发者工具 / 隐私工具 / 邮箱客户端（高端、极简、克制动效）

---

## 1) 视觉基线（必须遵守）

### 色彩（以现有 CSS 变量为准）

本项目已有轻/暗两套配色，统一使用以下变量（来源：`frontend/src/styles/global.css`）：

- 背景：`--app-bg`（浅色：#f6f7fb / 暗色：#0b1220）
- 表面（玻璃卡片）：`--app-surface`、`--app-surface-2`
- 边框：`--app-border`、`--app-border-strong`
- 主文本：`--app-text`
- 次文本：`--app-text-muted`
- 品牌色：`--app-brand`（浅色：#2563eb / 暗色：#60a5fa）
- 强调色：`--app-brand-2`（浅色：#0ea5e9 / 暗色：#38bdf8）

**原则：**
- 以“清晰可读”为第一优先；浅色模式文本对比度必须 ≥ 4.5:1。
- 高级感来自留白、层级与克制阴影，而不是高饱和渐变或厚重阴影。

### 字体

- UI 字体：`--app-font-sans`（Fira Sans + 系统字体回退）
- 等宽字体：`--app-font-mono`（Fira Code + 回退，用于 ID、Token、代码片段）

### 圆角 / 阴影 / 玻璃质感

- 圆角：`--app-radius`、`--app-radius-sm`
- 阴影：`--app-shadow-sm`、`--app-shadow-md`
- 玻璃：`backdrop-filter: var(--app-backdrop)`（只用于面板/卡片；列表项不要滥用）

### 动效（克制、优雅）

- 缓动：`--app-ease`
- 时长：`--app-duration-fast`（160ms）/ `--app-duration`（220ms）

**动效规则：**
- 优先使用颜色/边框/阴影的过渡，避免“缩放导致布局跳动”。  
- 尊重 `prefers-reduced-motion`：用户偏好减少动效时应关闭不必要动画。

---

## 2) 信息架构（IA）与交互（页面级）

### 顶层导航（Header）

- 仅保留三个主入口：主页（收发）、用户、管理员。
- 右侧只放“工具类图标按钮”：主题、语言、GitHub（可配置显示）。
- **必须**有明确的当前页高亮（active state）。
- **建议**加入跳转主内容（Skip Link），提升键盘可用性。

### 页面内部导航（Tabs）

- Tabs 的角色是“任务切换”，不承载过多说明文字。
- 标签文案：中文优先 2–4 字（如「收件」「发件」「写信」「外观」），英文优先单词（Inbox/Outbox/Compose）。
- 横向 Tabs（top/bottom）优先使用更“高级”的分段/条形样式；纵向 Tabs（left/right）保持紧凑。
- **建议**支持深链接：URL Query 反映当前 Tab（便于分享/刷新保持状态）。

### 邮件阅读体验（核心）

- 桌面端：左侧列表 + 右侧内容，默认选中第一封（保持现有行为）。
- 移动端：列表 + 底部抽屉阅读，避免跳转页面。
- 列表项信息密度：只保留必要字段（主题 / 时间 / 发件人 / 关键提取），ID 放弱化位置或详情区。

---

## 3) 文案规范（简洁、可扫读）

- 避免「设置」「管理」等冗余后缀，除非歧义。
- 按钮用“动词 + 结果”：如「刷新」「复制」「解绑」「清空收件」。
- 提示信息一句话讲清楚，不堆叠解释；需要解释时用次文本（depth=3）或折叠展开。

---

## 4) 禁止项（Anti-pattern）

- ❌ 用 Emoji 当 UI 图标（可在文档中使用，但 UI 里统一用 SVG 图标）
- ❌ 过度动效（长时间流动背景、频繁闪烁、高频弹跳）
- ❌ 低对比文字 / 过浅的次文本（浅色模式尤其容易“显灰”）
- ❌ Hover 用 scale 导致布局抖动
- ❌ 交互无反馈（可点击元素必须有 hover / active / focus 状态）

---

## 5) 交付前检查清单

- [ ] 浅/暗模式下关键文字均清晰可读
- [ ] 可点击元素具备 hover + focus-visible
- [ ] Tabs/路由支持刷新后保持当前视图（至少主页与用户页）
- [ ] 移动端无横向滚动条，抽屉/弹窗不遮挡关键操作
- [ ] `prefers-reduced-motion` 生效（不强制动效）
- [ ] Responsive: 375px, 768px, 1024px, 1440px
- [ ] No content hidden behind fixed navbars
- [ ] No horizontal scroll on mobile
