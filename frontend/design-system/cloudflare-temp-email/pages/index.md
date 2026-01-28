# 主页（Index）— 设计说明

## 核心目标

- 主任务优先：**收件 / 发件 / 写信** 3 个动作在视觉上最突出。
- 次任务收敛：账户、外观、自动回复、Webhook、附件、关于保持可达，但不抢主任务注意力。
- 交互“可分享”：URL Query 反映当前 Tab（`?tab=`），同时兼容既有的 `?mail_id=` 查询逻辑。

## 信息结构

### 顶部信息区（AddressBar）

- 展示当前邮箱地址（可切换、可复制）。
- 右侧只放 1–2 个轻量操作：地址管理（弹窗）、复制（按钮）。
- 登录入口文案保持简洁：1 句错误提示 + 登录表单 + 「用户登录」入口。

### 主内容区（Tabs）

建议 Tab 文案（中文 / 英文）：

- `mailbox`：收件 / Inbox
- `sendbox`：发件 / Outbox
- `sendmail`：写信 / Compose
- `accountSettings`：账户 / Account
- `appearance`：外观 / Appearance
- `auto_reply`：自动回复 / Auto Reply（若开启）
- `webhook`：Webhook / Webhook（若开启）
- `s3_attachment`：附件 / Attachments（若开启）
- `about`：关于 / About（若开启）

## 邮件体验

- 列表项：主题为主、元信息为辅（时间 / 发件人 / 关键提取）；ID 弱化显示。
- 工具栏：默认模式只放“常用动作”，批量模式只放“批量相关动作”，避免同屏堆叠。

