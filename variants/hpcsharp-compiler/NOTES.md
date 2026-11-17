# hpcsharp-compiler 变体（仓库专用）

这个子目录保留 `backlog-workflow` skill 在 **hpcsharp-compiler** 仓库里的最后状态。
它不是替代根目录 `SKILL.md` 的另一份标准，而是同一 skill 的一个仓库本地化变体。

## 来源

- 源仓库：`git@gitlab-fw.internal.unity.cn:vfx-team/hpcsharp-compiler.git`
  （本地 clone：`/Users/zhongyuan/Documents/u3d/gitlab/hpcsharp-compiler`）
- 源路径：`.agents/skills/backlog-workflow/SKILL.md`
- 引入提交：`867b3bdd8` —— `docs(skill): 新增 backlog-workflow skill（Backlog 读写唯一真相，与 tdd-workflow 同源跟踪）`
- 提取时状态：`532e6a31f`（2026-11-17 重置前的 master HEAD）；该文件自 `867b3bdd8` 起未再改动
- 校验：242 行 / 13178 字节，md5 `f439b053c439d9371f92d5d72057f883`

## 与根 SKILL.md 的差异（约 15 行）

根 `SKILL.md` 是角色中立的通用版（占位符 `TASK-123`，版本门禁写「项目规定的已批准版本/构建标识」）。
本变体把门禁硬钉在该仓库的 Backlog fork 上：

- 版本门禁改为「`backlog --version` 输出必须含 `-thu.`」，不含即停手；
- 示例任务号改为 `THU-123` / `THU-124`。

除此之外正文一致。

## 为什么单独留档

源仓库在 2026-11-17 把 master 回退到 Backlog 引入之前（`f916612a5`），准备在
backlog-workflow 收口后重做一次 Backlog 迁移。此变体是该 skill 在源仓库内的最后状态，
避免随重置丢失；重做迁移时可从这里取回，或直接以根 `SKILL.md` 为基线重新本地化。
