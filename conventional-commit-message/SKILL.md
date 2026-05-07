---
name: conventional-commit-message
description: "Use when: generating or reviewing SVN commit messages or Git commit messages that must follow Conventional Commits 1.0.0, including short commit titles, multi-line commit bodies, breaking-change footers, staged-change summaries, and commit command suggestions for svn commit or git commit."
---

# Conventional Commit Message

Use this skill when the user asks for a commit message, SVN commit text, Git commit text, commit title, changelist summary, or commit command and wants the result to follow Conventional Commits 1.0.0.

## References

- Conventional Commits 1.0.0: https://www.conventionalcommits.org/zh-hans/v1.0.0/

## Core Format

Every commit message must use this header shape:

```text
<type>[optional scope][!]: <description>
```

Use an English lowercase `type` and optional English lowercase/kebab-case `scope`. The description may be Chinese when the repository team communicates in Chinese.

Examples:

```text
feat(api): 新增分页查询能力
fix(ui): 修复按钮禁用状态
chore(svn): 整理提交清单
```

For breaking changes, use either `!` in the header or a footer:

```text
feat(config)!: 调整配置格式

BREAKING CHANGE: 旧版配置格式不再兼容
```

## Type Selection

Prefer these types:

- `feat`: user-facing feature or new capability.
- `fix`: bug fix.
- `docs`: documentation-only changes.
- `refactor`: code restructuring without behavior change.
- `test`: tests or validation-only changes.
- `chore`: repository maintenance, cleanup, or non-product code.
- `build`: build system, package, assembly, dependency, or compiler configuration.
- `ci`: CI workflow changes.
- `perf`: performance improvement.
- `style`: formatting-only changes.
- `revert`: revert a previous change.

When a change mixes multiple unrelated types, recommend splitting the commit. If the user insists on one commit, choose the type that represents the primary shipped intent and mention the mixed scope in the body.

## Scope Selection

Choose a narrow scope from the affected module, feature, package, or workflow.

Good scopes:

- `api`
- `auth`
- `config`
- `data`
- `docs`
- `test`
- `ui`
- `svn`
- `git`
- `build`

Avoid vague scopes such as `misc`, `update`, `change`, or `work`.

## Workflow For SVN Or Git

1. Determine repository type if needed:
   - SVN: `svn status` and optionally `svn diff --summarize`.
   - Git: `git status --short` and optionally `git diff --name-only`.
2. Identify only the files that should be included in the commit.
3. Exclude unrelated, ignored, local tool, generated IDE, package-lock, or conflict files unless the user explicitly wants them committed.
4. Summarize the user-facing or engineering intent, not every file.
5. Produce the shortest valid message when the user asks for a short message.
6. Add a body only when it materially helps reviewers understand risk, exclusions, migration, validation, or follow-up.
7. Add footers only for references, reviewers, issue IDs, or breaking changes.

## SVN Guidance

SVN does not have staged changes by default, so be extra careful with file selection.

When suggesting an SVN commit command, prefer an explicit file list if the working copy contains unrelated changes:

```bash
svn commit path/to/file-a path/to/file-b -m "feat(scope): 描述"
```

If the commit message needs multiple lines, recommend an editor or a message file rather than stuffing long escaped text into `-m`.

Never assume changelist names such as `ignore-on-commit` are safe to submit. Treat them as excluded unless the user says otherwise.

## Git Guidance

When suggesting a Git commit command, distinguish between staged and unstaged changes.

For already staged changes:

```bash
git commit -m "feat(scope): 描述"
```

For unstaged changes, first suggest reviewing and staging the exact intended files:

```bash
git add path/to/file-a path/to/file-b
git commit -m "feat(scope): 描述"
```

Do not suggest `git add .` when unrelated or generated files may be present.

## Output Style

Default output should be compact:

```text
feat(scope): 简短描述
```

If the user asks for SVN and Git variants, use the same Conventional Commit message for both unless the commit content differs.

When giving commands, separate message and commands:

```text
提交信息：
feat(api): 新增分页查询能力

SVN：
svn commit <files> -m "feat(api): 新增分页查询能力"

Git：
git commit -m "feat(api): 新增分页查询能力"
```

## Review Checklist

Before finalizing a commit message, check:

- Header matches `<type>[scope][!]: <description>`.
- Type is lowercase and meaningful.
- Scope is specific and not noisy.
- Description is imperative/summary-like and concise.
- Breaking changes are marked with `!` or `BREAKING CHANGE:`.
- SVN/Git command advice does not accidentally include unrelated files.
