---
name: llm-workflow-maintainer
description: 'Maintain an LLM Wiki style Markdown workflow. Use when: LLM-Workflow, LLM Wiki, knowledge base ingest, meeting transcript summary, workflow lint, feature index update, publication scan, public/private boundary, bootstrap new workflow instance.'
argument-hint: 'Describe the workflow task, source path, private instance path, and whether public repository changes are allowed.'
user-invocable: true
disable-model-invocation: false
---

# LLM Workflow Maintainer

## Goal

Use the public `LLM-Workflow` protocol as the canonical workflow source while keeping runtime actions inside the current private instance or the appropriate public repository.

This skill is intentionally thin. It must not duplicate the full workflow document.

## Use When

- Ingesting a new source into a Markdown knowledge layer.
- Summarizing a meeting transcript into decisions, action items, risks, and open questions.
- Updating a feature or topic index.
- Linting a workflow instance for stale claims, broken links, orphan pages, or missing sources.
- Bootstrapping a new private workflow instance.
- Deciding whether content belongs in a public workflow repository or a private instance.
- Preparing public workflow or skill repository changes for publication.

## Canonical References

- Workflow repository: <https://github.com/Counull/LLM-Workflow.git>
- Canonical protocol: `LLM_WORKFLOW.md` in the workflow repository.
- Bootstrap checklist: `docs/BOOTSTRAP_NEW_INSTANCE.md` in the workflow repository.
- Publication guard: `docs/PUBLICATION_GUARD.md` in the workflow repository.

## Read Order

For a private instance task:

1. Instance `AGENTS.md`.
2. Instance index, usually under `02_index` or a localized equivalent.
3. Instance `README.md`.
4. Canonical `LLM_WORKFLOW.md`.
5. Relevant source, meeting, design, development, or testing pages.

For a public workflow repository task:

1. Repository `AGENTS.md`.
2. `LLM_WORKFLOW.md`.
3. `docs/PUBLICATION_GUARD.md`.
4. `docs/SKILLS_INTEGRATION.md`.
5. Files directly requested by the user.

## Routing Rules

| Content | Destination |
|---|---|
| Generic workflow rule | Public `LLM-Workflow` repository. |
| Reusable task procedure | Public `UniSkills` repository. |
| Raw source, transcript, or private document | Private instance only. |
| Project-specific decision or implementation detail | Private instance only. |
| Company, client, customer, or proprietary context | Private instance only. |
| Runtime skill definition | `UniSkills`, not `LLM-Workflow`. |

Never create a second full copy of `LLM_WORKFLOW.md` in a private instance. Link to the canonical workflow instead.

## Standard Operations

### Ingest

1. Identify the source and keep raw material in the private instance.
2. Extract durable facts, decisions, risks, open questions, and affected topics.
3. Create or update the appropriate knowledge-layer page.
4. Update the index.
5. Mark source confidence explicitly.

### Meeting Summary

1. Treat audio and transcripts as raw sources.
2. Preserve uncertainty in `transcript.md`.
3. Write `summary.md` with `Decisions`, `Discussion Tendencies`, `Action Items`, and `Open Questions`.
4. Update affected feature or topic pages when the meeting changes requirements, design, or acceptance criteria.

### Workflow Lint

Check for:

- Broken links.
- Stale or superseded claims.
- Missing index entries.
- Orphan pages.
- Missing source references.
- Private content in public repositories.
- Duplicated workflow documents.

### Publication Scan

Before committing to public repositories, scan for:

- Company, client, or project names.
- Private file paths or local machine paths.
- Proprietary requirements, code paths, diffs, issue IDs, branch names, or commit IDs.
- Meeting transcripts, meeting decisions, participant data, or sensitive audio references.
- Credentials, endpoints, account names, tokens, or environment details.

If unsure, keep the content private and ask for review.

## Output Requirements

- State which layer was updated: public workflow, public skill, or private instance.
- Mention every created or updated file.
- If a public repository was changed, report publication-scan result and Git status.
- If no files were changed, say so clearly.
