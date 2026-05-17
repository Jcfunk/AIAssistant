# Agent Guidance for AI-Assistant

## Project Facts
- **What**: Notepad++ dockable panel plugin (Windows DLL)
- **Language**: C++20, Win32 API, Unicode
- **Build**: MSBuild (`AI-Assistant.vcxproj`) or CMake (`CMakeLists.txt`)
- **No npm, no Node.js, no test framework, no CI workflows**

## Build Commands
```powershell
# MSBuild (primary)
msbuild AI-Assistant.vcxproj /p:Configuration=Release /p:Platform=x64 /m

# CMake (alternative)
cmake -S . -B build
cmake --build build --config Release
```

**DLL output**: `build/x64/Release/plugins/AI-Assistant/AI-Assistant.dll`

## Install to Local Notepad++
```powershell
scripts/install-npp-ai-plugin.ps1
```
Default: copies DLL from build tree to `C:\Program Files\Notepad++\plugins\AI-Assistant\`

## Package for Release
```powershell
scripts/package-npp-ai-plugin.ps1 -Platform x64
```
Generates zip + Plugins Admin JSON in `dist/`. Refuses to package if `.pdb` files are staged.

## Source Layout (5 compile units)
| File | Responsibility |
|------|---------------|
| `src/AIAssistant.cpp` | Plugin entry (`DllMain`), panel UI, settings dialog, prompt builder, context menu, i18n, threading |
| `src/shared/HttpClient.cpp` | WinHTTP wrapper — GET, POST, SSE streaming with callbacks |
| `src/shared/LLMApiClient.cpp` | Provider API calls — OpenAI, Gemini, Claude, Copilot, Local (Ollama-compatible) |
| `src/shared/SecureStorage.cpp` | DPAPI-encrypted API key storage under `%LocalAppData%` |
| `src/shared/SettingsStorage.cpp` | INI-based non-secret preferences under `%AppData%` |

## Critical Conventions
- **Single-turn only** — no conversation memory between requests. Do not add multi-turn state.
- **Secrets vs preferences are split** — DPAPI for keys (`SecureStorage`), INI for settings (`SettingsStorage`). Never mix.
- **API keys must be wiped** after use via `wipeString()`. Do not leave keys in long-lived buffers.
- **Gemini auth uses `x-goog-api-key` header** — never put API keys in URL query strings.
- **RichEdit20W** for chat display (not plain EDITTEXT).
- **Cross-thread UI updates** use `PostMessageW` with `WM_AI_RESPONSE_READY` / `WM_AI_STREAM_CHUNK`. Never touch UI from worker thread directly.

## Linked Libraries
`comctl32, advapi32, crypt32, shell32, shlwapi, winhttp, riched20`

## Vendored Headers
`vendor/notepadpp/` (plugin + docking API), `vendor/scintilla/include/`

<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **AIAssistant** (971 symbols, 2488 relationships, 86 execution flows). Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

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
