# kstack 设计草案

> 状态：草案，待确认。确认前不写任何 SKILL.md。

## 1. 为什么要做（修订旧决策）

2026-06-07 定下的方案是「纪律进 CLAUDE.md，流程整包追踪 mattpocock/skills，自己只放上游覆盖不了的东西，不 fork、不融合」。当时的底层判断是：**纪律会贬值，记忆会增值**。

kstack 不推翻这个判断，而是把它推到底：模型编码能力变强以后，贬值的不只是纪律，还有大量「怎么做」的百科式规则。emil 那套约 4000 行里，很大一部分是模型本来就会、只是偶尔偷懒的 how-to；matt 那套则绑定了他的 tracker / 阶梯流程，和我实际的交付方式（Claude 主循环 + 子代理 + dureview + stacked PR）只部分重合。真正增值、而且上游永远给不了的，是三样东西：

1. **我的口味**：UI 的审美底线、我认为「AI 味」的具体表现、我接受和拒绝的取舍。
2. **我的决策**：技术栈（TanStack / Next、Bun、Vite、Monorepo、FSD、DDD）以及它们在具体场景下怎么落地。
3. **我踩过的坑**：nmem 里的部署、发布、跨机器、平台差异等经验。

所以 kstack 的定位是：**一套以我的偏好和决策为主体、上游内容按需吸收的自有 skill 集**。它会**替代** matt + emil + pstack，而不是叠加在上面。

三套上游各取所长：

- **matt**（mattpocock/skills）：取它区分「事实」和「决策」的做法，以及 ADR / GLOSSARY 的维护方式。
- **emil**：取 UI 判断和精确数值，数值放进 `references/`。
- **pstack**（poteto，Cursor 插件）：只取能对付「过度规划、注意力消耗」的部分：
  - `architect`：先写调用方怎么用，再从用法推出类型；至少给出两个结构不同的草图；默认不设人工确认点。
  - 三条原则：`never-block-on-the-human`、`sequence-verifiable-units`、`foundational-thinking`（数据结构先行）。

  不收的部分：`arena`/`swarm` 的多模型并跑，因为我这里是 Opus 主循环 + Fable advisor，没有 grok 那类评审组；`poteto-mode` 的 23 个 playbook 也不收。

## 2. 入选标准

每一条候选内容都过一遍这道筛子：

| 问题 | 是 → | 否 → |
|---|---|---|
| 这是我的偏好、决策或项目约定吗？ | 进 SKILL.md 正文 | 继续问 |
| 不写的话，Opus 级模型会**稳定地**做错吗？ | 进正文，并写清 why | 继续问 |
| 需要精确数值或查表（色阶、缓动曲线、字号表）吗？ | 放 `references/`，按需读取 | **删掉** |

**按领域区分标准（2026-10-18 修订）：**

上面这道筛子只适用于我有生产经验的领域，比如后端、交付和工程流程。前端设计我还在学习期，过去的偏好和决策还在变化，不能当依据。所以前端设计以选定的老师 emil 为准，**不从 nmem 提炼口味**。等我形成稳定判断后，再用一个薄 skill 覆盖 emil 的个别规则。

这里的 emil 指本机 `~/.agents/skills` 下的 28 个 `emil-*`，它们来自我购买的 aiforui.dev 设计课程。这套是单独安装的，不在 `.skill-lock.json` 里，原样保留，不 fork、不改写。它和开源的 emilkowalski/skills（15 个 skill）不是同一套，不要安装后者，否则 `prototype`、`mobile-native` 等名字会撞车。

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
- **`design`**：先拿出草图，再挑战草图，取代原来的 grill 和 spec。（pstack architect、matt grilling 的「事实 / 决策」区分、harness/templates/task-brief.md）详见下方「流程桶的两个原则」。
- **`diagnose`**：难缠 bug 和性能回退的诊断循环。（diagnosing-bugs）
- **`review`**：按「仓库规范」和「相关 ADR」两条轴审查改动；涉及 UI 时自动加上 UI 审查这一维。（code-review、emil-ui-review）
- **`handoff`**：把当前会话压缩成另一个 agent 能接手的文档。（handoff）
- 对抗式审查（pstack `interrogate` 的思路）不单独成 skill，作为 `review` 的可选模式，只在设计有争议时打开。

#### 流程桶的两个原则

**一、先出草图，再挑战草图，不做开放式问答。**

grill 消耗注意力，根因在顺序：它先抛出一棵开放的决策树让人逐个回答，可很多决策在动手之前根本没法定。`design` 把顺序反过来：

1. 事实自己查：读代码、查 nmem、看 ADR。不拿事实问人。
2. 直接给出具体草图：先写调用方的用法，再写数据结构和类型签名，并给出一个结构不同的备选方案。
3. 可逆的决策自己定，写进草图并说明理由（这就是 CLAUDE.md 里的 state and proceed）。
4. 只有不可逆的决策或产品方向才问人，并且放在草图**之后**，以「待定问题」的形式列出，一般不超过三个。
5. 还没法决定的事，明确标成「留给实现阶段决定」，不要硬定。

挑战一个具体草图，比回答一串抽象问题省力得多。CLAUDE.md 里「work is large 时展开 fuller design 并确认」，指的就是这个 skill。

**二、功能的事实源只有一份：代码 + 测试。文档只记录决策。**

- **不写 spec。** 功能文档必然会和代码漂移，过时以后会误导 agent，还逼着仓库维护两份事实。
- **`design` 的产物是一份短 ADR**，在**做出决策时**写，不在实现之前写。ADR 只有四节：Problem / Decision / Alternatives（每个一行）/ Consequences。
- **ADR 带日期，并标注 Accepted 或 Superseded by ADR-NNNN 状态。** 它记录的是历史，不是现状，所以过时是被允许的；它回答的是「当时为什么这么定」，而不是「现在是什么样」。agent 只把 Accepted 状态的 ADR 当作约束。
- **任务拆分不入库。** 大任务拆成子代理 brief（沿用 harness/templates/task-brief.md）和 stacked PR。brief 用完即弃，留在 PR 描述里就够了。
- ADR 和 GLOSSARY 的维护规则放在 `domain-design` 里，`design` 只负责引用。

### 前端（Frontend）
- **设计（UI/UX）**：原样使用本机的 28 个 `emil-*`，kstack 不收录。
- **`frontend-architecture`**：已放弃，原因见验证记录。
- **和其他组的关系**：
  - emil 的 `ui-review` 是流程组 `review` 的 UI 维度；
  - emil 的 `prep-for-prod` 是交付组 `release` 的 UI 维度；
  - 这两处都直接引用 emil，不复制内容。
- **和 matt 重叠时以 emil 为准**：`prototype` 用 emil 的版本。

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
- grill-me、grilling、grill-with-docs、to-spec、to-tickets、implement-spec：由 `design` 和「只记录决策」的原则取代。
- pstack 的 poteto-mode、arena、swarm、recall、ketchup 等：依赖 Cursor 环境和多模型组合，或者已有更合适的替代（nmem、obelisk）。
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

### 验证记录

**2026-10-18，后端组**

- **做法**：给两个子代理同一个场景：Go + Postgres 的推荐奖励 webhook、新增 API 字段、header 改名、蓝绿部署。一个加载后端三个 skill，一个不加载，对比输出。
- **结论**：
  - 「事务内不调外部服务」「唯一约束做幂等」「expand/contract」「outbox」这些教科书规则，不加载 skill 也会做到，属于 no-op，已经删掉。
  - skill 的增量全部来自偏好和具体事故：
    - 按上下文划分状态所有权；
    - 字段名带单位；
    - 术语写进 CONTEXT.md；
    - ADR 只记决策，部署细节放 PR；
    - envelope 带 `schemaVersion`；
    - mock 用真实响应。
- **局限**：只跑了一个场景，而且这个场景和 skill 的素材（nmem 里的事故）同源。真正的检验要等下一个不同领域的真实任务。

**2026-10-18，前端口味草图作废**

我曾经从 nmem 和 PiGUI 里提炼过一份前端口味草图。它被否决了：前端还处于学习期，过去的选择不能当依据。文件已经删除，前端设计改为以 emil 为准。

**2026-10-18，frontend-architecture 放弃**

- **做法**：同一个场景（Web 控制台 + Electron + 后续 Expo，后端是已有的 Go 服务）做 A/B，两组都带上了全局 CLAUDE.md 的技术偏好。
- **结果**：不加载 skill 的一组也做到了这些：
  - 框架选择（后端已有时用纯客户端的 TanStack Router）；
  - FSD 切片；
  - 用 Steiger 做 lint；
  - 有第二个使用方才提升到 packages；
  - Expo 路由文件保持很薄；
  - 不把服务端数据复制进客户端 store。
- **差异**：只有一点，Electron 主进程按 DDD 组织。
- **结论**：CLAUDE.md 里的单行技术偏好已经足够驱动行为，这个 skill 是 no-op。前端组在 kstack 里因此为空。
- **推论**：写 skill 之前，先看 CLAUDE.md 里的一行规则是否已经覆盖了它。

## 7. 待确认

已确认：
- pstack 指 Cursor 里 poteto 的插件；
- 走替代路线；
- grill 改成「先草图后挑战」，spec 取消，改为只记决策的 ADR。

仍待确认：
1. pstack 的吸收范围（第 1 节）是否合适。
2. 流程桶的新形态，也就是 `design`、`diagnose`、`review`、`handoff` 这四个，有没有要增删的。
3. 前端口味：emil 之外，有没有你明确喜欢或讨厌的设计参照？这一条不影响后端桶先行。
