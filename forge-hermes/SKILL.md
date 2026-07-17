---
name: forge-hermes
description: 锻造——AI辅助产品开发流程。五阶段：需求澄清→架构设计→审查→TDD逐任务实现→归档。子agent自动派发审查和实现。
version: 1.0.0
author: AnHuoZhe
license: MIT
---

# 锻造 (Forge) for Hermes

你是锻造流程的编排者。用户说"锻造一个XXX"时，按五阶段推进。

核心流程定义见本仓库的 `CORE.md`。本文件只包含Hermes平台特定的调用方式。

## 平台特定规则

1. 阶段A和B在主对话完成（交互式，需要用户参与决策）
2. 阶段C和D用 `delegate_task` 派发子agent（干净上下文，自动运行）
3. 每个阶段的产出写入文件，不只在对话里说
4. 审查通过后直接进入实现，不再反复确认

## 阶段A：需求澄清

按CORE.md阶段A的模板和问题清单执行。产出 `forge-brief.md`，用户确认后进入阶段B。

## 阶段B：架构设计

按CORE.md阶段B的模板执行。产出 `forge-design.md`（含任务拆解），展示全文给用户确认后进入阶段C。

## 阶段C：审查

用 `delegate_task` 派发审查子agent：

```
delegate_task(
  goal="审查设计文档，按CORE.md阶段C的标准（五层切口+六维度+收敛规则）执行审查。",
  context="项目路径：{当前目录的绝对路径}\n请按以下步骤：\n1. read_file读取forge-brief.md和forge-design.md\n2. skill_view加载forge-product-reviewer技能（审查流程规范）\n3. read_file读取CORE.md（审查标准定义）\n4. 执行审查，产出forge-review-{N}.md\n5. 按收敛规则判断是否通过"
)
```

子agent返回后：必须改逐条修改设计，建议改判断是否采纳。未收敛则再派发（最多5轮）。

## 阶段D：TDD逐任务实现

从 `forge-design.md` 读取任务列表，逐个派发：

```
delegate_task(
  goal="实现任务：{任务描述}。按CORE.md阶段D的TDD要求执行。",
  context="项目路径：{当前目录的绝对路径}\n任务：{任务描述和预期产出}\n请按以下步骤：\n1. read_file读取forge-design.md确认接口\n2. skill_view加载forge-implementer技能（实现流程规范）\n3. read_file读取CORE.md（TDD标准定义）\n4. 严格TDD执行\n5. 完成后git diff确认改动"
)
```

每任务完成后确认diff和测试通过，再派发下一个。

## 阶段E：归档

按CORE.md阶段E的模板产出 `forge-summary.md`，git commit归档，告诉用户完成。
