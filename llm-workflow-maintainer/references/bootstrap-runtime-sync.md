# Bootstrap Runtime Sync

## Goal

初始化 LLM Wiki 知识库实例时，不只创建 Markdown 目录，还要同步准备运行时入口：runtime skills、LLM Wiki Sync agent，以及本机实例 registry。

## Architecture

| Layer | Responsibility | Storage |
|---|---|---|
| Canonical workflow | 通用协议、结构、发布边界 | Public `LLM-Workflow` repo |
| Runtime skills | 可复用操作流程、agent 模板、bootstrap 规则 | Skills repo / local skills root |
| User agent | VS Code 中可直接唤起的工作入口 | VS Code user `agents/` 或 workspace `.github/agents/` |
| Instance registry | 本机知识库实例 ID 到本地路径的映射 | `~/.llm-wiki/instances.json` |
| Private instance | 原始资料、需求、会议、项目知识页与索引 | 用户指定的私有目录 |

## Why Registry

本地知识库路径是机器相关信息，不应该进入公开仓库，也不适合写死到可云同步的 agent 模板。registry 用来承接这类本机状态：

- agent 可以通过实例 ID 找到私有知识库。
- 多个知识库实例可以共存。
- 换机器后重新初始化 registry 即可，不污染同步的 agent 配置。
- 公开 workflow 和 skills 仓库保持无私有路径。

## Registry Location

默认位置：

```text
~/.llm-wiki/instances.json
```

建议内容：

```json
{
  "schemaVersion": 1,
  "defaultInstanceId": "instance-id",
  "instances": [
    {
      "id": "instance-id",
      "name": "Instance Display Name",
      "path": "path/to/private-instance",
      "scope": "private",
      "canonicalWorkflowRepo": "https://github.com/Counull/LLM-Workflow.git",
      "skillsRepo": "https://github.com/Counull/UniSkills.git",
      "createdAt": "YYYY-MM-DD",
      "updatedAt": "YYYY-MM-DD"
    }
  ]
}
```

## Bootstrap Steps

1. Pull or update the runtime skills repository.
2. Install or update `llm-workflow-maintainer/agents/llm-wiki-sync.agent.md` into the VS Code user agents directory, unless the user requests a workspace-local agent.
3. Create or update `~/.llm-wiki/instances.json` with the new instance entry.
4. Create the private instance structure from the canonical workflow bootstrap checklist.
5. Write instance `AGENTS.md` with the instance identity, canonical workflow link, and public/private boundary rules.
6. Write or update the instance index entry.
7. Report changed files and whether public content was touched.

## Local Link Placement

Use this split:

- Machine-local absolute path: `~/.llm-wiki/instances.json`.
- Human-readable instance entry point: private instance `README.md`.
- Agent behavior and routing rules: `AGENTS.md` inside the private instance.
- Reusable bootstrap procedure and agent template: skills repo.

Avoid putting absolute paths in public repos or synced agent templates.

## Update Policy

- If the agent template changed, bootstrap may overwrite the installed agent only after checking whether the user has local edits.
- If a registry entry with the same `id` exists, update `path`, `updatedAt`, and repo URLs; preserve unknown fields.
- If multiple instances exist and no default is set, bootstrap should set the new instance as default only with user confirmation.
- If the target instance is inside a Git repo, registry remains outside that repo unless the user explicitly asks for a repository-local manifest.
