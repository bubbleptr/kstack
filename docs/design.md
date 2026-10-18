# kstack 设计草案

> 状态：草案，待确认。确认前不写任何 SKILL.md。

## 1. 为什么要做（修订旧决策）

2026-06-07 定下的方案是「纪律进 CLAUDE.md，流程整包追踪 mattpocock/skills，自己只放上游覆盖不了的东西，不 fork、不融合」。当时的底层判断是：**纪律会贬值，记忆会增值**。

kstack 不推翻这个判断，而是把它推到底：模型编码能力变强以后，贬值的不只是纪律，还有大量「怎么做」的百科式规则。emil 那套约 4000 行里，很大一部分是模型本来就会、只是偶尔偷懒的 how-to；matt 那套则绑定了他的 tracker / 阶梯流程，和我实际的交付方式（Claude 主循环 + 子代理 + dureview + stacked PR）只部分重合。真正增值、而且上游永远给不了的，是三样东西：

1. **我的口味**：UI 的审美底线、我认为「AI 味」的具体表现、我接受和拒绝的取舍。
2. **我的决策**：技术栈（TanStack / Next、Bun、Vite、Monorepo、FSD、DDD）以及它们在具体场景下怎么落地。
3. **我踩过的坑**：nmem 里的部署、发布、跨机器、平台差异等经验。

所以 kstack 的定位是：**一套以我的偏好和决策为主体、上游内容按需吸收的自有 skill 集**，替代而非叠加 matt + emil。

## 2. 入选标准

每一条候选内容都过一遍这道筛子：

| 问题 | 是 → | 否 → |
|---|---|---|
| 这是我的偏好、决策或项目约定吗？ | 进 SKILL.md 正文 | 继续问 |
| 不写的话，Opus 级模型会**稳定地**做错吗？ | 进正文，并写清 why | 继续问 |
| 需要精确数值或查表（色阶、缓动曲线、字号表）吗？ | 放 `references/`，按需读取 | **删掉** |

附加规则：

- `~/.claude/CLAUDE.md` 里已经常亮的闸门（设计先行、TDD、根因、证据、Trust no stale docs、复杂度自证）**一律不在 skill 里重述**，skill 只放流程和口味。
- 每条规则配一句 why；一条规则删掉后行为不变，就删。
- SKILL.md 正文目标 ≤150 行；超出部分拆进 `references/`。
- description 用英文写清「做什么 + 何时用 + 触发词」，因为它决定能否被正确加载。

## 3. 分层

```
~/.claude/CLAUDE.md, ~/.agents/AGENTS.md   纪律（常亮，单行命令式）
kstack/skills/                             流程 + 口味 + 技术决策（按需加载）
<project>/.agents/skills, CONTEXT.md, ADR  项目专属上下文（随项目走）
nmem                                       跨项目的经验与决策（kstack 的素材来源）
```

kstack 的内容会周期性地从 nmem 提炼：某条经验在 nmem 里反复出现、并且已经稳定，就升格成 skill 里的一条规则。

## 4. 骨架（候选，约 16 个）

分四个桶加一个路由。括号里是主要素材来源。

### 路由
- **`kstack`**：在不知道用哪个 skill 时给出选择和顺序。（ask-matt、emil-ask-emil）

### 流程（Flow）
- **`grill`**：拷问计划或设计，区分「事实自己查」和「决策问人」，结束前必须确认理解一致。（grill-me、grilling、grill-with-docs）
- **`spec`**：把讨论沉淀成 spec 并拆成带依赖边的任务简报，直接对接我的子代理 brief 格式和 stacked PR。（to-spec、to-tickets、harness/templates/task-brief.md）
- **`diagnose`**：难缠 bug 和性能回退的诊断循环。（diagnosing-bugs）
- **`review`**：按仓库规范和 spec 两条轴审查改动；UI 改动时自动加上 UI 审查这一维。（code-review、emil-ui-review）
- **`handoff`**：把当前会话压缩成另一个 agent 能接手的文档。（handoff）

### 前端（Frontend）
- **`frontend-architecture`**：FSD 在 TanStack Start / Next / Vite SPA / Expo / 原生下分别怎么切；路由、数据获取、状态归属的默认选择。（我的偏好 + nmem，**上游空白**）
- **`ui-craft`**：我的设计底线，覆盖层级、间距、字体、颜色、表面、动效、交互状态。正文只写判断和口味，数值进 `references/`。（emil 的 foundations、typography、color、surfaces、animations、ui-polish、forms、touch-and-accessibility 合并压缩）
- **`unslop`**：去除代码、界面、文案里的 AI 痕迹。（emil-unslop-code、emil-unslop-design、emil-unslop-writing）
- **`prototype`**：用多个真正不同的版本探索方向，或者用一次性代码验证状态模型。（matt prototype、emil-prototype）
- **`component-api`**：React 组件 API 设计，以及 shared/ui 组件注册到设计画廊的约定。（emil-component-design + PiGUI 经验）

### 后端（Backend，上游全部空白）
- **`domain-design`**：DDD 限界上下文怎么划、领域逻辑和传输层 / 持久层怎么隔离，以及 GLOSSARY / ADR 的维护。（domain-modeling、codebase-design 的 deep module 词汇 + 我的偏好）
- **`api-contract`**：Monorepo 里的共享契约包、校验、错误模型、版本演进。（**上游空白**）
- **`data-and-migrations`**：数据建模、迁移、事务边界、回填。（**上游空白**）

### 交付（Ship）
- **`ship`**：feature 分支 → ready PR → dureview → 合并后清理；stacked PR 用 `gh stack`；PR 正文写法。（matt pr + CLAUDE.md 的 Git 规则 + nmem 的 gh stack 经验）
- **`release`**：发布前检查和部署模式，包括 release 目录 + 符号链接的原子切换、前后端独立部署、SSR 健康检查。（emil-prep-for-prod + nmem 的 sub2api / FlatRouter 经验）

### 元
- **`write-skill`**：kstack 自己的写作规范，也就是第 2 节的入选标准。（emil-writing-skills、writing-for-agents）

### 有意不收的
- research、teach、wizard、to-questionnaire、retro、wayfinder、triage、resolving-merge-conflicts、design-vocabulary、engineering-vocabulary、marketing-pages、mobile-native、ask-lapse、borrow-a-brain、build-a-tool、get-creative、product-thinking：要么模型已经足够胜任，要么和全栈开发主线无关。需要时可以继续单独安装上游版本。
- obelisk、better-readme、kami、ego-browser：独立工具，和 kstack 无关，保持原样。

## 5. 仓库形态与安装

```
kstack/
  skills/<name>/SKILL.md
  skills/<name>/references/*.md
  docs/            中文设计文档、迁移记录
  README.md
```

- 通过 `npx skills add` 安装（和现有的 `.skill-lock.json` 走同一条管理路径），本地开发阶段用 symlink 到 `~/.agents/skills/`。
- 某个桶迁移完成后，卸载被它替代的上游 skill，避免触发词冲突（例如两个 prototype、code-review 和 ui-review 同时匹配）。

## 6. 迁移顺序

1. **后端桶**优先：上游空白，增量最大，素材主要来自 nmem。
2. **前端桶**：合并压缩 emil，确定我的 UI 口味。这一步需要我（Kieran）大量参与判断。
3. **流程 + 交付桶**：在 matt 的基础上改写，对接我现有的 Claude 主循环 + dureview + gh stack 流程。
4. 每个 skill 写完后，用一个真实任务跑一次，对比有无该 skill 时的行为差异；没差异就删。

## 7. 待确认

1. 「pstak」具体指哪一套？本机没有安装，nmem 里只有 gstack（Garry Tan）的分析记录。
2. 替代还是叠加：建议替代，按桶逐步卸载上游。
3. 骨架里的增删：哪些是你实际高频使用的，哪些可以砍。
4. 前端口味：emil 之外，有没有你明确喜欢或讨厌的设计参照（产品、站点、设计系统）？
