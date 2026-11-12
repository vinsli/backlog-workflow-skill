---
name: backlog-workflow
version: 1.1.0
description: >
  项目使用 Backlog.md 的标准化工作流：初始化探测、task 创建、Spec/Plan 生命周期、
  按可交付目标拆分子任务、子任务派工与 Notes 进度、父任务里程碑同步和收口。
  触发词：Backlog 初始化、创建 task、Backlog Spec/Plan、拆子任务、子任务进度、
  更新父任务、Backlog workflow、task closeout。
---

## When to Use

当工作需要被记录、规划、派工、交接或验收时使用本 skill。它是项目 Backlog
操作的唯一标准来源；技术循环、主 agent/Worker 的具体角色边界和代码验收由项目
自己的技术工作流或派工规范负责，本 skill 不绑定某个仓库的角色名称。

以下工作通常必须建 task：需要方案或决策、需要跨 session 恢复、需要交给 Worker、
有多个可交付目标、或需要留下验收证据。纯问题解答、一次性探索和明显机械的小改动
可以不建 task，但不得借此绕过已有 task 的生命周期。

## 不可违反的门禁

1. **先验证 CLI 版本**：每次 Backlog 读写前执行 `backlog --version`；输出必须符合
   当前项目规定的已批准版本/构建标识（例如项目明确要求 `-thu.` 时必须包含它）。
   不符合时立即停止，报告版本问题，禁止改用 PATH 中的其它 `backlog`。
2. **再验证仓库已初始化**：在仓库根目录执行 `backlog config list`，并用
   `backlog overview` 做一次只读探测。两者任一失败都视为未初始化或目录错误；
   停止 Backlog 操作，报告当前目录和错误，等待用户确认后才可执行
   `backlog init ...`。不要用直接查看/创建 `backlog/` 文件夹代替 CLI 判断。
3. **所有 Backlog 读写走 CLI**：任务、子任务、Notes、Spec、Plan、评论、文档、
   决策和状态都用全局 `backlog` 命令。禁止直接编辑 `backlog/` 下的 markdown。
4. **先读指南再写字段**：创建任务前读 `backlog instructions task-creation`；
   开始、计划、改状态、派工或写 Notes 前读
   `backlog instructions task-execution`；验收、Final Summary 或 Done 前读
   `backlog instructions task-finalization`。
5. **字段是事实源**：Plan、Notes、状态和 Final Summary 必须能让新 session 从
   Backlog、git 和 handoff 恢复工作，不依赖旧对话记忆。

### 初始化检查模板

```bash
cd <repository>
backlog --version                 # 必须符合项目规定的 pinned build
backlog config list               # 成功 = CLI 找到项目配置
backlog overview                  # 成功 = 项目可被只读访问
```

版本门禁或初始化探测失败时，不创建 task、不改 task、不直接修 markdown。若用户确认
要初始化，再使用 CLI 的 `backlog init`，初始化完成后重新执行三项检查。

## 生命周期总览

`Intake → Spec/Plan → Dispatch → Execute → Closeout` 是 Backlog 操作边界。
状态名以项目配置为准（例如 `To Do → In Progress → Done`）；不要用 label
伪造阶段或 heartbeat。

- **Intake**：搜索是否已有 task；没有才创建。只记录问题、动机、范围和验收意图，
  不写 speculative Implementation Plan。
- **Spec/Plan**：开始 task 后读全量上下文、确认依赖和约束，补齐 Spec，形成按交付
  目标组织的 Plan。Plan 草案先评审；涉及产品、架构或流程规则的任务，Reviewer
  通过且用户明确冻结后，才写入 `## Implementation Plan`。
- **Dispatch**：把冻结 Plan 拆为独立、可验收的子任务；每个子任务是一个有界交付包，
  绑定一个 Worker 和一份交接指针。
- **Execute**：Worker 只推进自己的子任务；过程、阻塞、已验证结论和证据指针写入
  子任务 Implementation Notes。父任务只由主 agent 更新里程碑。
- **Closeout**：主 agent 先验收子任务，再更新父任务；全部 AC 有证据后写 Final
  Summary 并将任务置为配置的终态。

冻结后的 Spec/Plan 变更是重规划：暂停派工，记录变更原因，重新评审并重新冻结；
不得静默扩大子任务范围。

## 创建 task 的标准

### 1. 先搜索、再决定结构

```bash
backlog search "<关键词>" --plain
backlog task list --exclude-status "Done" --search "<关键词>" --limit 20 --plain
```

找到同一目标的 task 就复用它；不要创建重复 task。一个目标一个父 task；有自然
独立交付点且共享同一目标时使用 subtasks：

```bash
backlog task create "<父任务标题>" \
  -d "为什么需要这项工作，以及未来执行者无法从代码恢复的背景" \
  --ac "可独立验证的结果 1" \
  --ac "可独立验证的结果 2" \
  --spec "必须交付的范围、约束和不做什么"
backlog task create -p TASK-123 "<子任务标题>" ...
```

跨目标、不同所有权或只存在先后关系的工作创建独立 task，并用 `--dep` 表达依赖，
不要强行塞进同一个父 task。创建前使用 `backlog task create --help` 确认当前 CLI
字段。

### 2. task 内容要求

- **Title**：说明交付目标，避免只写“调查一下”或实现细节。
- **Description**：写问题/触发原因、用户需要和范围背景；不要把验收标准或实现
  方案伪装成描述。
- **Acceptance Criteria**：每条是独立、可测试、面向结果的条件；包括文档、测试、
  证据和边界条件（若属于交付）。
- **Spec**：写“必须交付什么”、约束、兼容性和验收意图；需要明确范围时创建时就
  提供，或 task 开始时补齐。
- **Implementation Plan**：创建普通 To Do task 时禁止写；它属于执行阶段的已审
  方案。只有已经直接以活动状态创建的工作，CLI 才允许在创建时带 plan。
- **References/Docs**：加入实现所需的文件路径、设计文档或外部参考；不要用它们
  代替 Description/AC。

不要以“实现某个 helper”作为唯一 AC；代码结构是手段，用户可验证的交付结果才是
验收标准。创建完成后立即读取 task，确认 ID、状态、Spec 和 AC 正确。

## 开始 task：确定 Spec 与 Plan

开始已有 task 时按顺序执行：

```bash
backlog task view TASK-123 --plain
backlog task edit TASK-123 --status "In Progress" --assignee @<owner>
backlog task edit TASK-123 --spec "<缺失的交付范围、约束、验收意图>"
```

在写 Plan 前确认：依赖已满足或有明确阻塞、当前 task 没有重复实现、工作区单写者
条件成立、技术路由已经确定。Plan 必须回答：

1. 最终有哪些可交付目标，分别如何验收；
2. 每个目标需要哪些有序步骤和前置依赖；
3. 哪些步骤可拆为独立子任务、每项的停手条件和证据；
4. 哪些决定需要 Reviewer 或用户确认；
5. 如何在中断后凭 Backlog Notes、git 和 handoff 恢复。

Plan 应按“交付目标”而不是按文件列表拆分。一个子任务应在一次有界工作中完成，
拥有单一验收口径；太大的子任务继续拆，只有机械且不可独立验收的步骤才合并。

写入正式 Plan 前：

```bash
backlog instructions task-execution
backlog task edit TASK-123 --plan $'1. 交付目标 A：...；验收：...\n2. 交付目标 B：...；验收：...'
```

`--plan` 只能记录已确认的实施方案。若任务要求评审，先把草案交 Reviewer，向
用户报告取舍并取得冻结确认；否则不要把草案写成当前真相。Plan 变化使用
`--plan` 替换或 `--append-plan` 追加，并在 Notes 记录原因与重新评审结果。

## 按交付目标创建和派工子任务

冻结 Plan 后，主 agent 逐项创建子任务。每项至少包含：目标与范围、独立 AC、Spec
约束、对应父 Plan 节、停手条件、交接文件路径和验证命令。子任务创建仍遵守“先
版本门禁、先搜索、CLI-only”；创建后读取确认父 ID、子 ID 和依赖。

```bash
backlog task create -p TASK-123 "<交付目标>" \
  -d "本项范围、父 Plan 节引用、完成后对父任务的贡献" \
  --ac "本项完成后可独立验证的结果" \
  --spec "本项必须交付的约束和不做什么" \
  --ref ".handoff/<子任务>.md"
```

派工前逐项检查：

- 同一工作树至多一个 writer；前一个 Worker 已退出、树净、上一项软租约已销记；
- 子任务粒度有界（遵循项目约定的一项一 Worker 纪律）；
- 任务书含目标、验收标准、停手条件和交接指针；
- 子任务状态/Notes 已写入派工时间、Worker、当前租约和预期证据。

父任务只由主 agent 更新。Worker 不修改父任务、不新建平行子任务、不静默扩大范围；
需要改变范围时停手并向主 agent 请求重规划。

## 执行期间的 Notes 纪律

子任务 Notes 是过程真相和恢复入口。每个有意义的推进点（开始、阶段完成、验证、
阻塞、等待决定、准备交回）都追加一次简短记录：

```bash
backlog task edit TASK-124 --append-notes $'2026-11-12T12:00+08:00｜heartbeat\n阶段：Spec §2 的实现\n已验证：<命令> → <结果/证据指针>\n阻塞：无\n下一步：<动作>'
```

Notes 必须包含阶段、事实、证据指针、阻塞（无则写“无”）和下一步；不要写流水账，
不要把未经验证的推测写成事实。长日志放在可直查文件或 handoff 中，Notes 只留
结论和指针。发现异常时追加阻塞及请求裁决，禁止继续做超出 Plan 的工作。

父任务 Notes 只保留五类里程碑：派工、阻塞、子任务验收、重规划、收口。子任务的
详细过程不复制到父任务；父任务记录子任务 ID、commit/证据指针和下一步调度决定。

## 子任务完成与父任务更新

子任务完成不是“Worker 说完成”。主 agent 必须先取得终态证据：子任务状态可更新、
Worker 已退出、handoff 完整、相关 commit/测试/文档证据可直查、工作树符合单写者
纪律。然后按顺序：

1. 读子任务并核对每条 AC；必要时运行独立验收命令；
2. `backlog instructions task-finalization`，用 CLI 写子任务 Final Summary，
   勾选已验证 AC，并将子任务置为 `Done`（或当前配置终态）；
3. 给父任务追加里程碑 Notes：子任务 ID、验收结果、证据指针、是否可派下一项；
4. 只有父任务计划中的下一项具备依赖和单写者条件时才继续派工；
5. 全部子任务完成后，逐项核对父 AC，写父 Final Summary，勾选 AC，更新父任务终态。

示例命令（字段以当前 CLI help 为准）：

```bash
backlog task edit TASK-124 --check-ac 1 --final-summary $'交付：...\n证据：commit <sha>；命令 <cmd> → PASS'
backlog task edit TASK-124 --status Done
backlog task edit TASK-123 --append-notes $'里程碑｜TASK-124 已验收\n证据：<commit/测试/handoff>\n下一步：...'
```

如果 AC 不能由证据证明，不勾选、不置 Done；记录阻塞并回到主 agent/Review。最终
收口必须包含逐 AC 的命令、输出或文件:行、commit/任务指针，不能只写“已通过”。

## 中断恢复与验证

新 session 接管时按顺序读取：父任务及在飞子任务的 `backlog task view`、Notes、
`git status`/`git log` 和 handoff 文件。Backlog 状态、git 链和 handoff 是事实来源；
不要依赖旧 session 的压缩上下文、presence 或 pane 标签推断终态。

最小完成检查：

- `backlog --version` 输出符合项目规定的 pinned build；
- `backlog config list` 与 `backlog overview` 成功；
- task 有 Description、AC、必要 Spec，且创建阶段没有 speculative Plan；
- 正式 Plan 可追溯到评审/冻结（若任务要求）；
- 每个子任务有独立 AC、Notes、验收证据和状态；
- 父 Notes 只有里程碑，且每个完成子任务都已回填父任务；
- Final Summary 逐父 AC 有可直查证据；
- 所有变更均由 CLI 完成，git diff 中没有直接编辑 Backlog markdown 的工作流绕过。

## Pitfalls

- 使用未经项目批准的 CLI：不同构建可能丢失 Spec 字段；发现版本不对必须停手。
- 未初始化时直接创建任务或手动创建 markdown：先报告并等待用户确认 `backlog init`。
- 创建 task 就写 speculative Plan：先写 Intake/Spec，执行阶段再形成并冻结 Plan。
- 一个父任务塞入多个无法独立验收的交付目标：按目标拆子任务并表达依赖。
- Worker 只更新父任务或只在聊天里报告进度：过程必须写入对应子任务 Notes。
- 子任务完成只看 pane/presence/口头报告：必须有 handoff、退出、代码/测试/文档证据。
- 父任务 Notes 复制全部子任务日志：只保留派工、阻塞、验收、重规划、收口里程碑。
- 直接编辑 Backlog markdown、用 label 表达阶段、或静默扩大冻结 Plan：均视为流程违规。

## Verification

本 skill 的流程验证不是“文档存在”而已：用上面的初始化检查、task view、子任务 Notes
和逐 AC 证据逐项核验；发现任一门禁失败就停止下一步写入，并把错误/阻塞记录在
当前 task Notes 中。
