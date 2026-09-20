# Vibeflow Plus

Vibeflow Plus is a model-agnostic CLI coding agent built for **Spec-Driven Development (SDD)**: a session works from an explicit Goal and a logged Plan, not an open-ended chat loop with no record of what it was asked to satisfy.

Status: experimental developer preview — see [SAFETY.md](SAFETY.md) before pointing it at anything you can't afford to lose. See [AGENTS.md](AGENTS.md) for this project's own agent-facing conventions.

## What this is built on

Vibeflow Plus is built on top of [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) (`dsh`), an MIT-licensed, "everything-is-a-plugin" agent harness powered by [Cordis](https://github.com/cordiverse/cordis). Every part of the product — model adapters, tools, session log, the agent loop itself — is a swappable plugin composed from configuration.

Full attribution and third-party license details: [LICENSE](LICENSE), [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).

## What's different from upstream

The base harness is a general-purpose plugin engine with no opinion on workflow, UI, or branding. Vibeflow Plus is a specific product built on top of it. The target for several of these decisions is explicit: product-surface parity with Claude Code, not just the narrower Aider/qwen-code baseline the original harness assumed.

- **A Spec-Driven Development workflow** — sessions carry an explicit Goal and a logged Plan (`packages/goal`, `packages/plan`), not just an agent loop.
- **A git safety net and native `/undo`** (`git-safety-net`, `command-undo`) — every edit gets a pre-edit checkpoint on a dedicated ref; nothing upstream does this.
- **Persistent cross-session memory** with mandatory, fail-closed secret redaction before any write — researched against Claude Code, Cursor, Windsurf, Mem0, and Zep; none of them redact by default the way this does.
- **Fine-grained tool/command permissions** — an AST-based shell-command tokenizer and per-tool/per-command allow/ask/deny rules, replacing two coarse session-wide knobs.
- **`/clear` and `/cleared`** — reset a session's context on demand, or read back what the last clear removed, without losing anything from the durable log.
- **Named subagent roles** (Inspector, Coder, Tester) with genuinely distinct tool filters and depth limits, not persona-text relabeling of one generic subagent.
- **Provider-agnostic in practice, not just in config** — the upstream system prompt and defaults carried a fixed model identity for one provider; this fork ships with no provider pre-registered and no favored route.
- **Its own Web UI and visual identity**, following Skycastle's design system — see [Visual identity](#visual-identity).
- **A curated default skill set**, in place of the harness's general-purpose defaults — see [Project conventions](#project-conventions).

Full detail, including what was reused as-is versus genuinely new, and every phase's audited verification: [docs/product/PROJECT_HISTORY.md](docs/product/PROJECT_HISTORY.md).

## Run

### Run from source

This project has no published package yet — run from a repository checkout. Requires Node.js `^22.19.0` or `>=24.0.0` and `pnpm@11.7.0` (see `package.json`'s `engines`/`packageManager` fields). `pnpm` itself needs Node 22+ to run at all; on an older Node, `pnpm install` fails with an internal `node:sqlite` stack trace instead of naming the real cause, so check your version first:

```sh
git clone https://github.com/tBeltty/vibeflow-advanced.git
cd vibeflow-advanced
node scripts/check-node-version.mjs && pnpm install
pnpm run build
pnpm dsh web
```

If the version check fails, install Node 22+ first — via [nvm](https://github.com/nvm-sh/nvm) (`nvm install 22`), [fnm](https://github.com/Node-multi-version-manager/fnm) (`fnm install 22`), or Homebrew (`brew install node@22`, then add it to `PATH`) — and re-run the command above. `pnpm run build` prepares the repository artifacts. `pnpm dsh web` starts the Web UI at `http://127.0.0.1:3080` by default and opens it in the default browser. Pass `--no-open` to run the server without opening a browser.

## Design goals

- **Model-agnostic**: OpenAI-compatible APIs and local inference servers (vLLM, Ollama, LM Studio), with no preferred or default-tuned model. See [`docs/guides/LOCAL_MODELS.md`](docs/guides/LOCAL_MODELS.md) for copy-pasteable Ollama and LM Studio `settings.yaml` presets.
- **Precise, token-efficient file editing**: SEARCH/REPLACE block edits with fuzzy-match fallback — never a full-file overwrite for large files.
- **Codebase awareness**: a compressed, Tree-sitter-backed repo map injected into the system prompt, respecting `.gitignore`.
- **Resilient shell execution**: cross-platform, non-interactive, timeout-bound, with explicit confirmation for destructive commands.
- **Context discipline**: active window monitoring with background compaction before it becomes a problem.
- **Protocol support**: native MCP (stdio + SSE) and isolated subagents for heavy analysis/testing work.

## Visual identity

Vibeflow Plus follows Skycastle's own design system: dark-mode only, flat and sharp-cornered (no glassmorphism), Skycastle's flagship violet (`#7C3AFF`) as its single accent color. See [`.claude/skills/atmos-ui-stylist`](.claude/skills/atmos-ui-stylist/SKILL.md) for the enforced rules.

## Project conventions

This repo carries two sets of Claude Code skills under `.agents/skills/` (symlinked at `.claude/skills/` for tool discovery):

- **`dsh-*` skills**, inherited from DeepSeek Harness, for maintaining the underlying monorepo (CI reliability, doc standards, PR conventions, etc).
- **Vibeflow Plus's own skills** — `clean-architecture-validator`, `dependency-navigator`, `cicd-sentinel`, `atmos-ui-stylist`, `pwa-compliance-steward`, `auditor-executor-protocol`, `trash-collector` — enforcing this project's own architecture, CI/CD, and visual-identity discipline.

All code, comments, documentation, and commit messages in this repository are written in English.
