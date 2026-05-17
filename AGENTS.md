# Background Agents

Read-only sub-agents for parallel background delegation. All agents have `edit=deny`, `write=deny`, `bash={"*":"deny"}`.

## Agent Directory

| Agent | Model | Purpose | Use When |
|-------|-------|---------|----------|
| `code-explorer` | `Frank-26B-A4B` | Trace execution flows, find callers/callees, understand architecture | "How does X work?", "Find all callers of Y", "Trace the auth flow" |
| `api-researcher` | `auto` | Research external APIs, check specs, verify compatibility | "Research OpenAI-compatible API specs", "Check Ollama response format" |
| `security-auditor` | `auto` | Review security practices, audit credential handling, find vulnerabilities | "Audit all credential paths", "Check for memory leaks with API keys" |
| `doc-writer` | `auto` | Draft documentation, guides, README sections | "Write a CONTRIBUTING.md", "Draft usage guide for Local provider" |
| `build-specialist` | `auto` | Research build systems, CI/CD, packaging, GitHub Actions | "Research MSBuild caching", "Optimize artifact compression" |
| `plugin-researcher` | `auto` | Research Notepad++ plugin ecosystem, patterns, API changes | "Find similar dockable panel plugins", "Research N++ API changes" |
| `i18n-helper` | `auto` | Review translations, assist with localization | "Review Traditional Chinese strings", "Research N++ localization patterns" |

## Usage

```
delegate(prompt, "agent-name")
```

- Use `delegate()` for all agents (read-only, no file edits)
- Write detailed prompts — agents run in isolated sessions
- Results are persisted and retrievable via `delegation_read(id)`
- Never poll `delegation_list()` — wait for `<task-notification>`

## Model Selection Rationale

- **Mini models** (`auto`): Fast, cheap — ideal for research, exploration, simple analysis
- **Medium models** (`auto`): Balanced — good for documentation writing, structured output
- **High models** (`auto`): Strong reasoning — reserved for security audits, complex analysis

All models use the `cursor-acp` provider. Models are assigned per-agent in `opencode.json`.

<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **AIAssistant** (972 symbols, 2104 relationships, 86 execution flows). Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

> If any GitNexus tool warns the index is stale, run `npx gitnexus analyze` in terminal first.

## Always Do

- **MUST run impact analysis before editing any symbol.** Before modifying a function, class, or method, run `gitnexus_impact({target: "symbolName", direction: "upstream"})` and report the blast radius (direct callers, affected processes, risk level) to the user.
- **MUST run `gitnexus_detect_changes()` before committing** to verify your changes only affect expected symbols and execution flows.
- **MUST warn the user** if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- When exploring unfamiliar code, use `gitnexus_query({query: "concept"})` to find execution flows instead of grepping. It returns process-grouped results ranked by relevance.
- When you need full context on a specific symbol — callers, callees, which execution flows it participates in — use `gitnexus_context({name: "symbolName"})`.

## Never Do

- NEVER edit a function, class, or method without first running `gitnexus_impact` on it.
- NEVER ignore HIGH or CRITICAL risk warnings from impact analysis.
- NEVER rename symbols with find-and-replace — use `gitnexus_rename` which understands the call graph.
- NEVER commit changes without running `gitnexus_detect_changes()` to check affected scope.

## Resources

| Resource | Use for |
|----------|---------|
| `gitnexus://repo/AIAssistant/context` | Codebase overview, check index freshness |
| `gitnexus://repo/AIAssistant/clusters` | All functional areas |
| `gitnexus://repo/AIAssistant/processes` | All execution flows |
| `gitnexus://repo/AIAssistant/process/{name}` | Step-by-step execution trace |

## CLI

| Task | Read this skill file |
|------|---------------------|
| Understand architecture / "How does X work?" | `.claude/skills/gitnexus/gitnexus-exploring/SKILL.md` |
| Blast radius / "What breaks if I change X?" | `.claude/skills/gitnexus/gitnexus-impact-analysis/SKILL.md` |
| Trace bugs / "Why is X failing?" | `.claude/skills/gitnexus/gitnexus-debugging/SKILL.md` |
| Rename / extract / split / refactor | `.claude/skills/gitnexus/gitnexus-refactoring/SKILL.md` |
| Tools, resources, schema reference | `.claude/skills/gitnexus/gitnexus-guide/SKILL.md` |
| Index, status, clean, wiki CLI commands | `.claude/skills/gitnexus/gitnexus-cli/SKILL.md` |

<!-- gitnexus:end -->

## Architecture Documentation

`ARCHITECTURE.md` contains the authoritative codebase architecture derived from the GitNexus knowledge graph.

### Purpose

- **Rapid onboarding**: New agents or developers can understand the full system layout without reading every source file
- **Impact analysis context**: Before editing, cross-reference the functional areas and execution flows to understand what you're touching
- **Provider integration reference**: The provider support matrix and data flow summary show how each LLM provider connects
- **Mermaid diagram**: Visual overview of all component relationships (Notepad++ host → plugin → shared libraries → HTTP → LLM providers)

### When to Use

- Starting work on a feature that spans multiple modules — read the functional areas table first
- Debugging a provider-specific issue — check the execution flow traces and provider matrix
- Understanding where a symbol fits — the cluster-to-area mapping shows which functional area owns it
- Writing documentation — use as the canonical reference for system structure

### Regeneration

To regenerate after significant code changes:

```
# Re-index if needed
npx gitnexus analyze

# Then ask the agent to regenerate ARCHITECTURE.md from the knowledge graph
```

## Basic Memory — Knowledge Graph

This project uses **Basic Memory** for persistent knowledge management. Notes survive context compaction and form a searchable knowledge graph.

### Core Skills

| Skill | Use When |
|-------|----------|
| **memory-notes** | Creating or editing notes. Always search before creating to avoid duplicates. |
| **memory-tasks** | Multi-step work (3+ steps) that may outlast the context window. |
| **memory-schema** | New structured note types emerging (meetings, decisions, etc.). |
| **memory-lifecycle** | Marking work complete, archiving, status transitions. |
| **memory-ingest** | Processing meeting transcripts, pasted docs, external notes into structured entities. |
| **memory-reflect** | Consolidating recent activity into long-term memory (periodic or on demand). |
| **memory-defrag** | Memory files grown unwieldy, duplicates found, or weekly cleanup. |
| **memory-metadata-search** | Finding notes by frontmatter fields (status, priority, confidence, etc.). |
| **memory-research** | Researching external subjects (people, companies, technologies) via web search. |

### Note Structure

Every note has three sections:

```markdown
---
title: Note Title
tags: [tag1, tag2]
---

# Note Title

Body prose — background, reasoning, context.

## Observations
- [decision] Use REST over GraphQL for public API #api
- [risk] Third-party auth has no SLA
- [question] Should we support multi-tenancy?

## Relations
- implements [[API Specification]]
- depends_on [[Authentication System]]
```

- **Body**: Substantive prose for search retrieval and human understanding
- **Observations**: Atomic facts with `[category]` labels — fully searchable
- **Relations**: Wiki-links `[[Target]]` create knowledge graph edges

### Task Management

For multi-step work, create Task notes in `memory/tasks/`:

```yaml
---
title: Feature Name
type: Task
status: active
created: YYYY-MM-DD
current_step: 1
---
```

**Workflow**: Create → update `current_step` as you progress → set `status: done` when complete → archive.

**Before compaction**: Flush all active tasks with current state and context for resumption.

**After restart**: Search `search_notes("[status] active")` to find and resume work.

### Key Principles

- **Archive, never delete** — completed work is historical context
- **Search before creating** — use multiple query variations to find existing notes
- **Link liberally** — relations turn isolated notes into a knowledge graph
- **Externalize early** — if you think "I should remember this", write it down NOW
- **Categories are arbitrary** — invent `[category]` labels that fit your domain
- **Permalinks survive moves** — `memory://` URLs keep working after `move_note`

### Regeneration

To clean up or reorganize memory:

```
# Run defrag to split bloated files, merge duplicates, prune stale entries
# Run reflect to consolidate recent activity into long-term memory
# Run schema validate to check note conformance
```
