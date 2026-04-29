# Copilot Skills

这个仓库用于维护一组面向 GitHub Copilot 的本地技能定义，主要覆盖以下几类工作：

- 代码与文档的批判式审查
- LLM Wiki / Markdown 知识库工作流维护
- Unity 通用最小组件设计
- Unity 窄核心抽象设计

## 当前 Skills

### code-doc-roast

用于把代码现实和文档表述放在一起核对，集中指出职责混装、边界模糊、生命周期靠隐式约定维持等问题。

### llm-workflow-maintainer

用于维护 LLM Wiki 风格的 Markdown 工作流实例，处理 source ingest、会议转写整理、功能索引更新、workflow lint、公开/私有边界判断和发布前污染检查。

### unity-minimal-components

用于约束 Unity 中的小型可复用组件设计，强调单一职责、低耦合、极小 API 面，以及避免过早抽象。

### unity-narrow-core

用于设计比单文件 helper 稍大、但又明确小于框架级系统的 Unity 原生核心抽象，例如局部状态机核心或局部可逆命令核心。

## 仓库结构

```text
.
|-- code-doc-roast/
|-- llm-workflow-maintainer/
|-- unity-minimal-components/
`-- unity-narrow-core/
```

每个 skill 目录通常包含：

- `SKILL.md`：技能说明、触发条件和执行规则
- `agents/`：可选的子代理配置
- `references/`：可选的补充背景资料

## 使用说明

1. 在对应 skill 目录下维护 `SKILL.md`。
2. 需要附加背景资料时，放到该 skill 的 `references/` 中。
3. 需要专用代理配置时，在 `agents/` 中补充对应定义。

## 许可证

本仓库采用 MIT License，见 `LICENSE`。
