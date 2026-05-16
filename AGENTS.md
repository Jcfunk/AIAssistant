# Background Agents

Read-only sub-agents for parallel background delegation. All agents have `edit=deny`, `write=deny`, `bash={"*":"deny"}`.

## Agent Directory

| Agent | Model | Purpose | Use When |
|-------|-------|---------|----------|
| `code-explorer` | `gpt-5.4-mini-medium` | Trace execution flows, find callers/callees, understand architecture | "How does X work?", "Find all callers of Y", "Trace the auth flow" |
| `api-researcher` | `gpt-5.4-mini-medium` | Research external APIs, check specs, verify compatibility | "Research OpenAI-compatible API specs", "Check Ollama response format" |
| `security-auditor` | `gpt-5.4-high` | Review security practices, audit credential handling, find vulnerabilities | "Audit all credential paths", "Check for memory leaks with API keys" |
| `doc-writer` | `gpt-5.4-medium` | Draft documentation, guides, README sections | "Write a CONTRIBUTING.md", "Draft usage guide for Local provider" |
| `build-specialist` | `gpt-5.4-mini-medium` | Research build systems, CI/CD, packaging, GitHub Actions | "Research MSBuild caching", "Optimize artifact compression" |
| `plugin-researcher` | `gpt-5.4-mini-medium` | Research Notepad++ plugin ecosystem, patterns, API changes | "Find similar dockable panel plugins", "Research N++ API changes" |
| `i18n-helper` | `gpt-5.4-mini-medium` | Review translations, assist with localization | "Review Traditional Chinese strings", "Research N++ localization patterns" |

## Usage

```
delegate(prompt, "agent-name")
```

- Use `delegate()` for all agents (read-only, no file edits)
- Write detailed prompts — agents run in isolated sessions
- Results are persisted and retrievable via `delegation_read(id)`
- Never poll `delegation_list()` — wait for `<task-notification>`

## Model Selection Rationale

- **Mini models** (`gpt-5.4-mini-*`): Fast, cheap — ideal for research, exploration, simple analysis
- **Medium models** (`gpt-5.4-medium`): Balanced — good for documentation writing, structured output
- **High models** (`gpt-5.4-high`): Strong reasoning — reserved for security audits, complex analysis

All models use the `cursor-acp` provider. Models are assigned per-agent in `opencode.json`.
