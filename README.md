# 锻造 Forge for Hermes

> 原版 [Forge](https://github.com/AnHuoZhe/forge) 是 Codex 插件。本项目是专门为 Hermes Agent 移植的独立版本——用 `delegate_task` 替代 Codex 的 dispatch subagent，其余流程保持一致。

把模糊需求锻造成可交付产品的 AI 辅助开发流程。核心定义（流程、审查体系、上下文工程机制）在 [CORE.md](./CORE.md)，Codex 版与 Hermes 版共用。

## 与原版的关系

| | Forge (Codex) | Forge for Hermes |
|---|---|---|
| 运行平台 | Codex 桌面版 | Hermes Agent |
| 子agent机制 | dispatch a subagent | delegate_task |
| 技能格式 | Codex SKILL.md | Hermes SKILL.md |
| 流程 | 七阶段 | 七阶段（相同） |
| 审查体系 | 小审+架构审查+任务审查+中审+大审 | 小审+架构审查+任务审查+中审+大审（相同） |
| 上下文工程 | 渐进式披露+状态栏+澄清持久化+复审模式 | 渐进式披露+状态栏+澄清持久化+复审模式（相同） |

功能等价，适配不同平台。

## 前提条件

- Hermes Agent 已安装（`hermes` 命令可用）
- 有 API key（推荐 DeepSeek，便宜；或其他 provider）
- 会基本 git 操作

## 安装

把这三个目录复制到 Hermes 技能目录：

```
# Windows
复制到 C:\Users\<你的用户名>\AppData\Local\hermes\skills\

# Mac / Linux
复制到 ~/.hermes/skills/
```

最终目录结构：

```
~/.hermes/skills/
├── forge-hermes/              # 编排器
│   └── SKILL.md
├── forge-product-designer/    # 设计方法论（渐进式披露参考）
│   ├── clarify.md
│   ├── design.md
│   └── task-breakdown.md
├── forge-product-reviewer/    # 审查器
│   ├── SKILL.md
│   ├── cut-table.md
│   ├── architecture-review.md
│   ├── task-review.md
│   ├── module-review.md
│   ├── system-review.md
│   └── review-focus.md
└── forge-implementer/         # 实现器
    ├── SKILL.md
    └── debug-mode.md
```

`CORE.md` 放在这四个技能目录的上级（或项目可访问的公共位置），供子 agent 读取核心标准定义。子 agent 按 brief 的"加载文件"段在各自技能目录下查找专业文件（渐进式披露）。

## 使用

1. 在终端进入项目目录（或空目录）
2. 启动 Hermes：`hermes`
3. 加载技能：`/skill forge-hermes`
4. 说"锻造一个XXX"开始

## 流程

锻造分七个阶段（状态机名）：

**clarify** 需求澄清 — Hermes 和你对话，把模糊想法变成明确需求，产出 features.md（含澄清结论）
**design** 架构设计 — 基于需求设计架构，拆模块定接口，产出 architecture.md + tasks.json
**review** 架构审查 — 自动派发审查子 agent，多轮对撞直到收敛，产出架构审查报告
**build** 任务审查与逐模块实现 — 审查任务拆解，逐个模块派发实现子 agent，先写测试再实现，每任务小审
**module_review** 中审 — 模块完成锁定，自动派发模块级审查
**system_review** 大审 — 全部模块完成，自动派发系统级审查
**done** 归档 — 记录项目总结、时间线和教训

clarify 和 design 是你和 Hermes 对话完成。review、build、module_review、system_review 自动派发子 agent 完成。中审和大审由锁定条件强制触发，不能跳过。

## 上下文工程特性

- **渐进式披露**：子 agent 不全量加载技能，按 brief 的"加载文件"段按需加载专业文件，降低 token 成本
- **状态栏**：brief 携带运行状态（进度/连续失败/审查轮次/是否复审），子 agent 获得运行时感知
- **澄清持久化**：clarify 的核心结论写入 features.md，design 从文件恢复上下文，不依赖对话历史
- **复审模式**：审查未通过重审时，跳过全量扫描聚焦上一轮必须改项，提高复审效率

## 注意

- 审查阶段最多 5 轮，通过标准严格（必须改=0、建议改不增、无新严重担忧）
- 每个实现任务都是 TDD（测试先行），必须亲眼看到测试失败
- 中审/大审由流程条件自动锁住，用户无法跳过
- git commit 格式：中文，"第X版，做了什么"

## 许可

[PolyForm Noncommercial License 1.0.0](https://polyformproject.org/licenses/noncommercial/1.0.0/)（与 Forge 一致，见 [LICENSE](./LICENSE) 和 [LICENSE_CN.txt](./LICENSE_CN.txt)）

非商用许可：可自由使用、修改、分发，但不得用于商业目的。
