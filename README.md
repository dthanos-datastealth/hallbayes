# Berry

## Workflow verification playbooks

Want the “with vs without hallucination detector” experience? Start here:

- `docs/workflows/README.md` — index of workflow playbooks
  - Search & Learn
  - Generate Boilerplate/Content
  - Inline Completions
  - Greenfield Prototyping
  - RCA Fix Agent

Each playbook includes a **maximally contrasting** worked example (❌ vibes vs ✅ evidence + verifier).

Berry runs a local MCP server with a safe, repo‑scoped toolpack plus verification tools (`detect_hallucination`, `audit_trace_budget`).

Berry ships a single MCP surface: **classic**.

Classic includes:
- Verification tools (`detect_hallucination`, `audit_trace_budget`)
- Run & evidence notebook tools (start/load runs, add/list/search spans)

See `docs/MCP.md` and `docs/workflows/README.md`.

Berry integrates with Cursor, Codex, Claude Code, and Gemini CLI via config files committed to your repo.

## Supported verifier backends

Berry’s current verification method requires token logprobs (Chat Completions-style `logprobs` + `top_logprobs`).

Supported today:
- `openai` (default): OpenAI-compatible Chat Completions endpoints with logprobs (OpenAI, OpenRouter, local vLLM, or any compatible `base_url`)
- `gemini`: Gemini Developer API `generateContent` with token logprobs via `logprobsResult` (when enabled for the model)
- `vertex`: Vertex AI `generateContent` (Gemini models) with token logprobs via `logprobsResult`
- `dummy`: deterministic offline backend for tests/dev

Not supported yet:
- Anthropic (OpenAI-compat layer ignores `logprobs`)

## Quickstart

1) Install (from this repo):

```bash
pipx install -e .
```

Fallback:

```bash
pip install -e .
```

2) In the repo you want to use:

First, set your verifier API key (recommended):

```bash
berry setup
```

Then install repo-scoped MCP config files:

```bash
berry init
```

Optional: enable strict verification gates for that repo:

```bash
berry init --strict
```

3) Reload MCP servers in your client.

Optional: register Berry globally (user-level configs) so you don't have to commit repo files:

```bash
berry integrate

# macOS .pkg installers may also deploy system-managed configs:
#   berry integrate --managed-only
```

4) Use a prompt/workflow (Search & Learn (verified), Generate Boilerplate/Content (verified), Inline completion guard, Greenfield prototyping, RCA Fix Agent).

## Docs

- `docs/USAGE.md` — task‑oriented guides
- `docs/CLI.md` — command reference
- `docs/CONFIGURATION.md` — config files, defaults, and env vars
- `docs/MCP.md` — tools/prompts and transport details
- `docs/PACKAGING.md` — release pipeline (macOS pkg + Homebrew cask)

## Tests

```bash
pytest
```

---

## Claude Code Plugin (this fork)

This fork (`dthanos-datastealth/hallbayes`) extends Berry with native **Claude Code marketplace plugin support**. Where Codex CLI auto-injects Berry's workflow prompts via MCP, Claude Code requires them as skills — this fork closes that gap by porting all six Codex workflows as Claude Code SKILL.md files and packaging the whole thing as an installable marketplace plugin.

### What this fork adds

| | Original (`leochlon/hallbayes`) | This fork |
|---|---|---|
| Claude Code integration | Via `berry init` (writes `.mcp.json` to repo) | Native marketplace plugin — install with `/plugin install` |
| Workflow prompts in Claude Code | Not available (Claude Code doesn't auto-invoke MCP prompts) | Six workflow skills — one per Codex prompt |
| MCP server startup | Requires `pipx install -e .` first | Auto-starts via `uvx` — no pre-install needed |
| Configuration | Interactive `berry setup` CLI | `/berry-configure` slash command inside Claude Code |
| Codex CLI / Cursor / Gemini CLI | Unchanged | Unchanged |

### Installing

**1. Add the marketplace:**
```
/plugin marketplace add dthanos-datastealth/hallbayes
```

**2. Install the plugin:**
```
/plugin install berry@dthanos-datastealth-hallbayes
```

Claude Code clones the repo into `~/.claude/plugins/cache/berry-marketplace/berry/`, registers six workflow skills, and starts the MCP server automatically via `uvx`. No `pipx install` or `berry init` needed.

**3. Configure:**
```
/berry-configure
```

This interactive command walks through choosing your verifier backend, writing `~/.berry/mcp_env.json` with the correct credentials, and setting `allowed_roots` so Berry can read your project files for evidence collection.

**4. Restart Claude Code** to apply the configuration.

### Verifier backends

Berry scores claims using token logprobs from a verifier model. Any backend with logprobs support on `/v1/completions` works.

| Backend | Configuration | Notes |
|---------|--------------|-------|
| **Local llama.cpp** | `OPENAI_BASE_URL=http://127.0.0.1:8080/v1`, `OPENAI_API_KEY=none` | Best for privacy. Works with llama-server, Ollama, vLLM, LM Studio. |
| **OpenAI** | `OPENAI_API_KEY=sk-...` | Most reliable logprobs support. |
| **Gemini** | `GEMINI_API_KEY=...` | Google AI Studio key. Free tier available. |
| **Other OpenAI-compatible** | `OPENAI_BASE_URL=...`, `OPENAI_API_KEY=...` | OpenRouter, Groq, Fireworks, any `/v1/completions` endpoint with logprobs. |
| **Anthropic** | Not supported | OpenAI-compat layer drops `logprobs`. |

Configuration lives in `~/.berry/mcp_env.json`. Use `OPENAI_BASE_URL` (not `OPENAI_API_BASE`) — Berry uses the openai SDK v1.x.

### File access for evidence collection

Berry's `add_file_span` tool reads local files as evidence but requires explicit permission. Set `allowed_roots` in `~/.berry/config.json`:

```json
{
  "allowed_roots": ["/Users/yourname/repos"],
  "allow_web": false,
  "allow_exec": false,
  "allow_write": false,
  "audit_log_enabled": true
}
```

`/berry-configure` sets this interactively.

### Workflow skills

Six Berry Codex workflow prompts are available as Claude Code skills:

| Skill | What it does | Use when |
|-------|-------------|----------|
| `berry-search-and-learn` | Answers questions with evidence citations; every factual sentence requires a span | Researching APIs, understanding unfamiliar code |
| `berry-generate-boilerplate` | Generates tests/docs/config with a verified design-intent trace | Writing test scaffolding, config, migrations |
| `berry-inline-completion-guard` | Reviews a tab-complete via a micro-trace before accepting | A completion looks suspicious or touches critical paths |
| `berry-greenfield-prototyping` | Separates Facts (cited), Decisions, and Assumptions during prototyping | New projects, exploring architectures |
| `berry-rca-fix-agent` | 6-phase evidence-first RCA; verifies ROOT_CAUSE before writing fix | Debugging failures, incident investigation |
| `berry-plan-and-execute` | Produces a verified dry-run plan with approval gate before execution | Multi-step implementations, refactors |

### Updating

```
/plugin marketplace update berry-marketplace
/plugin update berry@dthanos-datastealth-hallbayes
```

Then restart Claude Code.
