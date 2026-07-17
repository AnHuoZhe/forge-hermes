# 锻造 Forge for Hermes

> 原版 [Forge](https://github.com/AnHuoZhe/forge) 是 Codex 插件。本项目是专门为 Hermes Agent 移植的独立版本——用 `delegate_task` 替代 Codex 的 dispatch subagent，其余流程保持一致。

把模糊需求锻造成可交付产品的AI辅助开发流程。

## 与原版的关系

| | Forge (Codex) | Forge for Hermes |
|---|---|---|
| 运行平台 | Codex 桌面版 | Hermes Agent |
| 子agent机制 | dispatch a subagent | delegate_task |
| 技能格式 | Codex SKILL.md | Hermes SKILL.md |
| 流程 | 五阶段 | 五阶段（相同） |
| 审查体系 | 精审+通审 | 精审+通审（相同） |

功能等价，适配不同平台。

## 前提条件

- Hermes Agent 已安装（`hermes` 命令可用）
- 有API key（推荐DeepSeek，便宜；或其他provider）
- 会基本git操作

## 安装

把这三个目录复制到Hermes技能目录：

```
# Windows
复制到 C:\Users\<你的用户名>\AppData\Local\hermes\skills\

# Mac / Linux
复制到 ~/.hermes/skills/
```

最终目录结构：

```
~/.hermes/skills/
├── forge-hermes/
│   └── SKILL.md
├── forge-product-reviewer/
│   └── SKILL.md
└── forge-implementer/
    └── SKILL.md
```

## 使用

1. 在终端进入项目目录（或空目录）
2. 启动Hermes：`hermes`
3. 加载技能：`/skill forge-hermes`
4. 说"锻造一个XXX"开始

## 流程

锻造分五个阶段：

A. **需求澄清** — Hermes和你对话，把模糊想法变成明确需求，产出 forge-brief.md
B. **架构设计** — 基于需求设计架构，拆模块定接口，产出 forge-design.md
C. **审查** — 自动派发审查子agent，多轮对撞直到通过，产出审查报告
D. **TDD实现** — 逐个任务派发实现子agent，先写测试再实现
E. **归档** — 记录项目总结和教训

阶段A和B是你和Hermes对话完成。阶段C和D是自动派发子agent完成。

## 注意

- 审查阶段最多5轮，通过标准严格（必须改=0、建议改不增、无新严重担忧）
- 每个实现任务都是TDD（测试先行）
- git commit格式：中文，"第X版，做了什么"

## 许可

MIT
