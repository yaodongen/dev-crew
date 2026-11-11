# dev-crew

[English](./README.md) | **简体中文**

dev-crew 把一个编程 agent 会话变成一位 manager：它按带关卡的流程推进一个正式的开发需求，把具体工作交给程序员 subagent，把 review 交给另外的、全新的 subagent。

**需求澄清 → 方案设计 → 方案 review → 你审批 → 实现 → 代码 review → 测试 → 变更总结 → 部署 dev**

它是一个纯粹的 [Agent Skill](https://agentskills.io)：一个 `SKILL.md` 加两个参考文件，没有代码，没有依赖。

> 状态：早期版本。面向 Claude Code 编写；支持 skill 和 subagent 的其他 agent 也应该可以使用。

## 为什么需要它

编程 agent 在较大的需求上出问题的方式很固定，dev-crew 针对其中四种：

| 问题 | dev-crew 的做法 |
|---|---|
| 一个 agent 看不出自己工作里的问题 | 方案和代码各由一个全新的 subagent review，它没见过作者的推理过程 |
| agent 会跳步（"看起来没问题，继续"） | manager 不写代码，只管流程；每一步都有关卡，满足并记录后才能进入下一步 |
| agent 会做没人要求的改动 | 实现完成后，manager 对照已批准的方案检查真实的 `git diff`，把方案外的改动逐条列给你，由你决定保留还是还原 |
| "已完成""测试通过"只是口头说法 | 测试必须有真实的命令输出存档，manager 还会自己重跑关键命令 |

## 安装

任选一种；用两种方式都装会出现两份同样的 skill。

**Claude Code plugin：**

```
/plugin marketplace add yaodongen/dev-crew
/plugin install dev-crew@dev-crew
```

**Claude Code 和其他 agent，通过 [skills CLI](https://github.com/vercel-labs/skills)：**

```bash
npx skills add yaodongen/dev-crew --global --agent claude-code   # Claude Code
npx skills add yaodongen/dev-crew --global --agent codex         # Codex
npx skills add yaodongen/dev-crew --global --agent '*'           # 所有检测到的 agent
```

去掉 `--global` 就只安装到当前项目。

**手动安装（Claude Code）：**

```bash
git clone https://github.com/yaodongen/dev-crew ~/.claude/skills/dev-crew
```

## 使用

用任何语言对 agent 说：

```
用 dev-crew 做这个需求：订单页增加导出 CSV
```

```
Use dev-crew for this task: add CSV export to the orders page
```

继续一个中断的需求：

```
继续 dev-crew 的 export-csv
```

manager 会用你的语言回复，过程文件也用你的语言写。

dev-crew 适合需要设计和评审的功能开发、重构、较复杂的 bug 修复。一行的小改动直接让 agent 做就行。

## 流程

```mermaid
flowchart LR
    S1[1 需求澄清] --> S2[2 方案设计] --> S3[3 方案 review]
    S3 --> S4{{4 你审批}}
    S4 --> S5[5 实现] --> S6[6 代码 review] --> S7[7 测试]
    S7 --> S8{{8 变更总结}} --> S9{{9 部署 dev 并演示}}
```

| # | 步骤 | 谁来做 | 关卡 |
|---|---|---|---|
| 1 | 需求澄清 | 程序员 A 提问；manager 有出处才回答，否则问**你** | 没有未决问题；验收标准可逐条验证 |
| 2 | 方案设计 | 程序员 A | `design.md` 完整 |
| 3 | 方案 review | 全新的程序员 B，最多 3 轮；manager 逐条裁决 | 没有未处理的阻塞意见 |
| 4 | 审批 | manager 把方案讲给**你** | 你明确批准。在此之前不动任何代码 |
| 5 | 实现 | 程序员 A，严格按方案 | 报告实现完成 |
| 6 | 代码 review | 全新的程序员 B 对照方案审 diff，最多 3 轮 | 没有未处理的阻塞意见 |
| 7 | 测试 | 程序员 A 测试；manager 重跑关键命令 | 每条验收标准都有真实输出佐证 |
| 8 | 变更总结 | manager 读真实 diff 后向**你**汇报 | 每项方案外改动都有你的决定 |
| 9 | 部署和演示 | 经**你**确认后 manager 部署到 dev，逐条演示验收标准 | 你确认演示通过 |

关卡是强制的。后面的步骤发现问题时，流程回到对应的前面步骤，其后的步骤重新走一遍。

### 在哪里改代码

开始前，manager 按项目已有的分支命名习惯拟一个分支名，然后：

- 当前目录有未提交的改动时，直接为这个需求建一个独立的 git worktree，并告诉你名字和位置，你的改动和需求的改动不会混在一起。
- 当前目录干净时，问你一次：建 worktree、在当前目录新建分支、还是就在当前分支上做。

### 涉及多个仓库的需求

方案仍然只写一份、审批一次，其中明确写出各仓库之间的约定。之后实现并行进行，每个仓库一名程序员；每个仓库的改动再由另一名全新的 reviewer 审查。测试时把所有改动合在一起验证。

### 它留下什么

过程文件保存在你仓库的 `.dev-crew/<需求>/` 下：需求、方案、每一轮 review 及 manager 的裁决、测试报告、变更总结，以及记录关卡的 `status.md`。会话中断后可以从这份记录继续。

dev-crew 不会自行 commit、合并或清理，这些都由你决定。

## 它是怎么设计的

```
dev-crew/
├── SKILL.md                 manager 的工作手册
└── references/
    ├── briefs.md            manager 派给程序员 subagent 的任务说明
    └── templates.md         过程文件和汇报的模板
```

设计基于大模型的三个行为特点：

- **独立上下文才有独立判断。** 看过作者推理过程的 reviewer 倾向于认同它，所以这里的 reviewer 只拿到产物。
- **manager 不干活，流程才不会丢。** 上下文里塞满实现细节的会话会忘记流程规则，所以 manager 的上下文保持很小。
- **文件是唯一的事实来源。** 每个结论都落盘，程序员在任何一步都可以换成全新的 subagent，不依赖一段很长的对话。

## 要求和限制

- 需要 git 仓库。变更总结依赖 diff。
- 需要 agent 支持 subagent。不支持时 skill 会告诉你并退化为单 agent 执行，这样就没有独立 review 了。
- 它比直接让 agent"做一下"更费 token 和时间，换来的是 review 和变更审计。

## License

[MIT](./LICENSE)
