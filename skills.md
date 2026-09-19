---
name: collab-doc-editor-design
description: 设计与搭建「类飞书文档 / Notion / Confluence / 语雀」协同文档产品与富文本编辑器的方法论——覆盖产品定位、信息架构、功能地图、块系统原则、编辑器内核选型（ProseMirror / Tiptap / BlockNote / Slate / Lexical / Quill / Univer）、Yjs/CRDT 协同架构、服务端领域模型、依赖清单与分阶段路线。用户提到富文本编辑器、块编辑器、在线文档、知识库、Wiki、实时协同编辑、CRDT、AFFiNE / AppFlowy / Docmost / Outline 对比，或想做"自己的飞书文档 / Notion"时务必使用本 skill——即使只是问"该选哪个编辑器""协同怎么做"这类局部问题。
---

# 协同文档产品 & 富文本编辑器设计

本 skill 把一次完整的开源生态调研（40 个仓库，2026-09）沉淀为可复用的决策框架。目标是让 Claude 在面对"做一个类飞书文档"这类需求时，不从零思考，而是按固定流程产出结构化、有依据的方案。

## 工作流

按顺序走完 5 步；用户只问局部问题时，直接跳到对应步骤，但输出前用步骤 1 的两三个问题快速校准上下文。

1. **澄清定位**（问清再答）：目标端（Web / 桌面 / 移动）、是否需要实时多人协同、部署方式（SaaS / 私有化）、团队编辑器经验、许可证约束（能否接受 GPL/MPL/收费扩展）。
2. **产品设计**：套用下方「产品设计框架」，先定信息架构与功能地图，再定块系统与非功能指标。
3. **技术选型**：用「选型决策表」给结论，并说明放弃项的原因。
4. **架构设计**：套用「架构蓝图」，给分层图、协同数据流、领域模型、依赖清单。
5. **落地路线**：给 P0–P3 分阶段交付，并把产品决策逐条映射到架构含义。

需要具体数据（Star 数、License、各项目依赖栈、架构细节）时读 `references/ecosystem-analysis.md`，按其目录跳到对应章节即可，不必整篇读。

## 核心结论（先记住这些）

- **编辑器内核事实上收敛到 ProseMirror 系**。Tiptap、BlockNote、Novel、Milkdown、Outline、Docmost 全部基于它；Schema 约束 + Transaction 模型 + 最成熟的 IME/Android 处理是原因。
- **协同层事实标准是 Yjs（CRDT）**。服务端用 Hocuspocus 起步，规模化后可换 Rust 实现（yrs / y-octo），协议不变，前端零改动。这是选 CRDT 而非 OT 的核心理由。
- **最贴近"团队可自己维护的类飞书文档"的参考实现是 Docmost**（Tiptap + Yjs/Hocuspocus + NestJS + PostgreSQL，全 MIT/Apache 依赖）；**最值得研究的数据模型是 AFFiNE/BlockSuite**（Block 与 CRDT 一体化），但不建议直接复用其 Lit 生态。
- **不要用**：Slate 系做主编辑器（中文 IME / Android 历史包袱）、Quill / Editor.js（块级嵌套与协同能力不足）、Flutter 路线（除非移动端优先）、CKEditor / TinyMCE（GPL + 协同收费）。

## 产品设计框架

产出顺序固定：定位 → 信息架构 → 功能地图 → 核心流程 → 块系统原则 → 非功能指标 → 产品→架构映射。

**定位表**至少回答五个问题：核心场景、目标用户（付费/使用/管理三角）、替代对象、差异化、明确不做什么。

**信息架构**默认采用：

```
Workspace → Space（知识库）→ Page（无限层级树）→ Block → Inline
横切：收藏/最近访问 · 评论/通知/版本 · 权限/分享/群组 · 搜索/标签/双链
```

**功能地图**按 P0（能写能协同）/ P1（团队可用）/ P2（差异化）三列，覆盖：编辑（基础块、交互）、协同、组织、权限、搜索、导入导出、端、管理后台。P0 基础块固定为：段落、标题、有序/无序/任务列表、引用、分割线、代码块、图片、表格，加 `/` 菜单、Markdown 快捷输入、浮动工具栏、拖拽排序。

**块系统五原则**：一切皆块且有稳定 ID；样式是属性不是类型；复杂块（多维表格、画板）自成一体、独立状态；渐进披露（默认只有输入框）；Markdown 是快捷键不是存储格式。

**非功能指标**给具体数值，默认基线：万字文档首屏 < 1.5s、5,000 块流畅、协同 P95 < 200ms、并发 ≥ 50 人、断网不丢数据、中文主流输入法零丢字。

**产品→架构映射表**必须给出，每条产品决策对应一条架构含义（例：划词评论 → 锚点 = 块 ID + Yjs RelativePosition）。完整示例见 `references/ecosystem-analysis.md` 第六节。

## 选型决策表

| 需求 | 推荐 | 备注 |
|---|---|---|
| Notion/飞书风格块编辑器，React，要快 | BlockNote 或 Novel | BlockNote 为 MPL-2.0，改其源码需开源该部分 |
| 完全自定义 UI，Vue 或多框架 | Tiptap | Pro 扩展（评论、版本、AI、docx）收费 |
| 长期演进、团队有编辑器经验 | ProseMirror 直接用 | 参考 Outline 的封装 |
| React + shadcn 全套 UI | Plate | 继承 Slate 底层风险 |
| 可访问性优先 / Meta 生态 | Lexical | 中文资料少 |
| 文档 + 表格 + 幻灯片统一底座、分页排版 | Univer | 纯文档场景过重 |
| 简单富文本输入框 | Quill | Delta 模型，不做块编辑 |

协同：默认 Yjs + Hocuspocus；需要服务端读懂文档做权限/审计/AI 时用 yrs/y-octo；只有强中心权威审计需求才考虑 OT（Etherpad/CKEditor 路线）。

## 架构蓝图

分层固定为：接入层 → 应用层 → 编辑器层（Block 模型 / 扩展 / UI / 内核）→ 协同层（Y.Doc ↔ y-prosemirror ↔ Provider，Awareness，UndoManager，y-indexeddb 离线）→ 数据访问层；服务端：网关 → API 服务 + 协同服务 + 异步 Worker → PostgreSQL / Redis / S3 / 全文检索。完整 ASCII 图在 `references/ecosystem-analysis.md` 7.1 节，直接复用并按用户上下文改名。

**编辑器 Schema**：`doc > blockGroup > blockContainer+`，container 带 `id` 与 `props`；复杂块做原子节点，内部状态放独立 `Y.Map`，用 NodeView 渲染。

**协同数据一致性六条**：
1. `pages.ydoc BYTEA` 存 Yjs 二进制，是唯一真相源。
2. `onStoreDocument` 防抖 2–5s 落库；Worker 定期快照作版本历史。
3. 同一钩子内投影出 ProseMirror JSON 与纯文本，供 API / 搜索 / AI。
4. 权限走页面级 + 空间级 ACL，`onAuthenticate` 返回 `readOnly`；不做块级权限。
5. 客户端 `y-indexeddb` 离线持久化。
6. 协同服务可后期 Rust 化，协议不变。

**领域模型**：Workspace → Member / Space → Page(tree) → {ydoc, content_json, text, PageSnapshot, Comment→Reply(anchor: block_id+range), Attachment} / Group / Permission(subject: user|group, object: space|page, level)。

**默认依赖清单（全 MIT/Apache）**：`@tiptap/core@3` `@tiptap/react` `@tiptap/pm` 或直接 `prosemirror-*`、`prosemirror-tables`、`prosemirror-markdown`、`lowlight`；`yjs` `y-prosemirror` `y-indexeddb` `@hocuspocus/provider`；React 19 + Vite + Zustand/Jotai + TanStack Query；NestJS + Kysely/Prisma + `@hocuspocus/server` + `extension-redis` + BullMQ；PostgreSQL 16 / Redis 7 / MinIO / Meilisearch（可选）；Electron 或 Tauri。

## 分阶段路线（默认模板）

| 阶段 | 目标 | 交付 |
|---|---|---|
| P0 0–2 月 | 能写、能协同 | 基础块、Yjs 协同、页面树、空间权限 |
| P1 3–4 月 | 团队可用 | 评论+@、版本历史、全文搜索、Markdown 导入导出、分享链接 |
| P2 5–6 月 | 差异化 | 多维表格、画板嵌入、模板、通知、移动端壳 |
| P3 6 月+ | 规模化 | 协同服务 Rust 化、AI、审计日志、SSO |

## 输出格式

给完整方案时用 Markdown 文件，章节顺序：产品设计框架 → 选型结论（含放弃理由）→ 架构蓝图（分层图 + 协同数据流 + 领域模型 + 依赖清单）→ 产品→架构映射 → 分阶段路线。局部问题直接在对话中用表格回答，不要生成文件。引用 Star 数或依赖版本时注明数据日期（参考文件为 2026-09 快照），并提示以仓库 lockfile 复核。

## 常见陷阱

- 把 Markdown 当存储格式：会丢失块属性与协同锚点；只把它当输入快捷键与导入导出格式。
- 在 ProseMirror ↔ Yjs 之外再维护一份"业务状态"：所有文档内容必须在 Y.Doc 内，否则协同必然不一致。
- 直接用 Tiptap Pro 的评论/版本扩展做核心功能：授权成本高且不可控，评论与版本历史应自研（锚点 + 快照方案见上）。
- 块级权限：复杂度远超收益，用页面级覆盖即可。
- 忽视中文 IME：所有 InputRule 在 compositionend 后触发；任何自定义 NodeView 都要在微软拼音 / macOS 拼音 / 搜狗下实测。
