# kstack

Kieran 的个人 skill 集，面向前后端全栈开发，取代了此前安装的 mattpocock/skills、emil 设计课程 skill 和 pstack。

- 设计与取舍：[docs/design.md](docs/design.md)，包括入选标准和每一组的验证记录。
- skill 源文件：`skills/<name>/SKILL.md`。按需读取的参考资料放在 `skills/<name>/references/`。
- skill 正文用英文写，文档用中文写。
- 纪律闸门（设计先行、TDD、根因、证据）写在全局规则里（`~/.agents/AGENTS.md`，Claude Code 用 `~/.claude/CLAUDE.md`，源文件都在 [harness](https://github.com/bubbleptr/harness) 仓库），所有工作都适用，skill 不重复这些内容。

## Skill 一览

「调用方式」一栏：**自动**表示模型会根据 description 自行加载；**手动**表示只有你输入 `/<name>` 才会加载，平时不占上下文。

### 流程

| skill | 调用方式 | 做什么 | 什么时候用 |
|---|---|---|---|
| `design` | 自动 | 先拿出具体草图：调用方用法、数据结构、一个结构不同的备选方案、已决定和推迟的事项，以及最多 3 个问题，再让你挑战。大的或涉及布局的工作放在可评论的 Doc（按轮回答，定稿后压成摘要）或画布上。不写 spec，ADR 只记决策 | 需求较大、范围模糊、涉及布局摆放，或有多种架构可选时 |
| `diagnose` | 自动 | 线上事故先查最近改了什么、先读错误文本，必要时先回滚；搭建能稳定复现的命令，排除低级原因，列出排序后的假设；按证据等级下结论 | bug、事故、偶发失败、性能回退 |
| `review` | 自动 | 按「仓库规范」和「Accepted ADR 与意图」两条轴分开审查；涉及 UI 时加跑 UI 审查，发布前可做全量检查；分拣 dureview 等自动审查的发现 | 审查分支、PR 或未提交的改动；收到审查评论；上线前检查 |
| `handoff` | 自动 | 交接文档、给子代理的派发简报、核验 worker 声称的「完成」、接手进行中的工作 | 换会话、派子代理、跨机器、续作 |

### 后端

| skill | 调用方式 | 做什么 | 什么时候用 |
|---|---|---|---|
| `backend` | 自动 | 领域：限界上下文、状态归属、分层、`CONTEXT.md` 和 ADR。契约：单一事实源、错误码、在边界做鉴权、安全地修改契约。数据：生产库陷阱、外部调用前后的事务、账本、迁移和发布顺序 | 新增后端概念或状态，改接口、消息或请求头，改表、写迁移，或涉及计费和余额 |

### 前端（从 emil 设计课程精简而来）

| skill | 调用方式 | 做什么 | 什么时候用 |
|---|---|---|---|
| `ui-design` | 自动 | 层级、排版、OKLCH 颜色与对比度、表面（圆角、阴影、边框）、图标、落地页，以及去除界面 AI 痕迹 | 搭建或调整界面外观；界面看起来不对、廉价，或一眼像 AI 做的 |
| `ui-motion` | 自动 | 要不要动、缓动曲线、时长、弹簧、进出场、减弱动效、动画性能 | 加动效或审查动效 |
| `ui-build` | 自动 | 收尾细节、布局稳定、交互状态与焦点、表单、触摸/键盘/读屏、移动端网页、渲染与加载性能 | 把能用的界面做到顺手、不卡、手机上像原生 |
| `component-api` | 自动 | 可复用组件的 props 设计，以及设计系统文档 | 新建或重构共享组件，写 DESIGN.md |
| `prototype` | 手动 | 做几个真正不同的版本，放在实时切换器后面让你挑 | 方向未定、想看几个选项时 |

### 交付

| skill | 调用方式 | 做什么 | 什么时候用 |
|---|---|---|---|
| `pr` | 自动 | 合入 main 之前：提交规范、开 PR 前要跑的检查、PR 正文模板、`gh stack` 的坑、等 CI 和读结果、合并后清理 | 提交、开 PR、推送、等 CI、合并 |
| `ship` | 自动 | 合入 main 之后：按可部署单元划定发版范围，处理版本、tag 和发版说明，验证产物，部署服务器（蓝绿、原子 release 目录），确认线上实际在跑什么 | 发版、打 tag、部署、回滚 |

### 跨领域

| skill | 调用方式 | 做什么 | 什么时候用 |
|---|---|---|---|
| `unslop` | 自动 | 去掉代码和文案里的 AI 痕迹 | 清理 agent 写的代码，开 PR 前，写面向读者的文字 |
| `setup` | 手动 | 每个仓库跑一次：写中文 `docs/agents/issue-tracker.md`，在根规则文件加 `## Agent skills` 指针，补齐 5 个 label；只支持 GitHub，已有 label 只补不删 | 新仓库第一次用 GitHub Issues、milestone 或 label 规划前 |
| `kstack` | 手动 | 路由：主流程、各 skill 索引、重叠时选哪个 | 不确定该用哪个 skill 时 |
| `write-skill` | 手动 | kstack 的写作规范：判断该不该存在、结构、description 写法、A/B 验证、落地步骤 | 新增或修改 kstack 的 skill |

## 安装

把每个 skill 软链接到两个目录：`~/.agents/skills` 供 Codex、Pi、Grok 读取，`~/.claude/skills` 供 Claude Code 读取（只在装了 Claude Code 的机器上需要，没装的机器把命令里的 `~/.claude/skills` 删掉）。仓库克隆到哪个目录都行。

```bash
mkdir -p ~/.agents/skills ~/.claude/skills
for d in skills/*/; do n=$(basename "$d"); for t in ~/.agents/skills ~/.claude/skills; do ln -sfn "$PWD/$d" "$t/$n"; done; done
```

软链接指向这个仓库，所以拉取更新后立即生效。新增、改名或删除 skill 时，要同步更新 `kstack` 路由和本 README。
