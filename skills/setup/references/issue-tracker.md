# Issue tracker: GitHub

本仓库的计划和待办都是 GitHub Issues，用 `gh` 操作。Issue 正文就是 spec；`docs/roadmap.md`（如有）只写方向和顺序，并链接到对应的 milestone。

## 基本操作

- **建 issue**：`gh issue create --title "..." --body-file -`，多行正文用 heredoc 从 stdin 传入。
- **读 issue**：`gh issue view <N> --comments`。需要结构化字段时用 `gh issue view <N> --json number,title,body,labels,milestone,comments`。
- **列 issue**：`gh issue list --state open --milestone "<title>" --label "<label>"`；需要正文时加 `--json` 和 `--jq`。
- **评论**：`gh issue comment <N> --body "..."`。
- **改 label**：`gh issue edit <N> --add-label "..."` / `--remove-label "..."`。
- **关闭**：做完用 `gh issue close <N> --reason completed`；不做用 `gh issue close <N> --reason "not planned"`，不要建 `wontfix`。
- **拆子任务**：`gh issue create --parent <N> ...`，或 `gh issue edit <parent> --add-sub-issue <child>`。
- **阶段内排序**：用 GitHub 原生 blocked-by。`gh issue edit <child> --add-blocked-by <blocker>`（gh 2.103+）；旧版用 API，`issue_id` 要 blocker 的 database id，不是 `#number`：
  `gh api --method POST repos/{owner}/{repo}/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>`。
  查 database id：`gh api repos/{owner}/{repo}/issues/<blocker> --jq .id`。

`gh` 在仓库目录里会自动推断 repo；不在仓库目录时加 `-R owner/repo`。

## 计划：milestone 与 backlog

- **milestone 只给有终点的阶段**，比如 `M3 在哪都能连`、`iOS 原生替换`。milestone 描述写阶段目标或验收标准，不写任务清单。
- 建 milestone：`gh api --method POST repos/{owner}/{repo}/milestones -f title="<title>" -f description="<goal>"`。
- 阶段做完就关闭 milestone（`gh api --method PATCH repos/{owner}/{repo}/milestones/<number> -f state=closed`），同时改 `docs/roadmap.md` 里的「当前阶段」。PiNomad 出现过 M6/M7 已关但 roadmap 仍写「现在」。
- **没有 milestone 的 open issue 就是 backlog**，这是正常状态；不要为了装 backlog 建一个「Backlog」milestone。
- backlog 用两个 label 分：
  - `later`：等明确需求再做。相关想法合进一个伞形 issue，正文用 checklist 列候选，不拆成一堆小 issue。
  - `tech-debt`：已知的技术债。
- **阶段内排序**用 blocked-by；**拆子任务**用 sub-issue。

## Spec 在 issue 正文

Issue 正文写：做什么、约束、已知坑、相关 ADR。计划变了就改正文；完成细节写进 PR，不回写 issue 或 roadmap。

原因是三者必须看同一份：planner 按 issue 做意图层 review，dureview 不读 spec，worker 的简报只引用 `#N` 而不复制正文。

PR 用 `Closes #N` 关闭对应 issue。checklist 类 issue 的部分进度在正文里勾选，不要提前关闭。

## Label

本仓库用下面五个 label。GitHub 默认 label 和历史 label 保留，不删除；新 issue 只用这五个。

| label | 什么时候用 |
|---|---|
| `ready-for-agent` | spec 写全了，planner 可以直接派给 worker |
| `ready-for-human` | 需要我本人动手或拍板：真机验证、账号操作、设计取舍 |
| `needs-triage` | 先记下来，还没想清楚 |
| `later` | 等明确需求再做；相关想法合进伞形 issue |
| `tech-debt` | 已知技术债 |

不要用 `needs-info`、`wontfix`、`wayfinder:*`。单人仓库没有外部报告者，`needs-info` 用不上；不做的 issue 用 `gh issue close --reason "not planned"`。

不用 GitHub Projects；状态用 label 表达。等有协作者、确实需要看板时，再考虑只加一个 Status 字段。

## 常用查询

- 当前阶段：`gh issue list --state open --milestone "<title>"`
- 可派给 worker：`gh issue list --state open --label ready-for-agent`
- 需要本人处理：`gh issue list --state open --label ready-for-human`
- 待分诊：`gh issue list --state open --label needs-triage`
- backlog：`gh issue list --state open --json number,title,labels,milestone --jq '.[] | select(.milestone == null) | {number, title, labels: [.labels[].name]}'`
