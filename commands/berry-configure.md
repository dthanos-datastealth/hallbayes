---
description: Configure Berry's verifier backend and file access permissions after installing the plugin
---

# Berry Configure

You are helping the user configure Berry after installing it as a Claude Code plugin. Berry is an evidence-first hallucination detection system. It needs a verifier — a language model that scores claims via token logprobs — before it can verify anything.

**Goal**: Write `~/.berry/mcp_env.json` (verifier credentials) and `~/.berry/config.json` (permissions) so Berry's MCP server connects to a real verifier on next restart.

## Step 1 — Show current state

Run these two commands and report the results clearly:

```bash
echo "=== mcp_env.json ===" && cat ~/.berry/mcp_env.json 2>/dev/null || echo "(not found)"
echo "=== config.json ===" && cat ~/.berry/config.json 2>/dev/null || echo "(not found)"
```

If `OPENAI_BASE_URL` is already set in `mcp_env.json` and `allowed_roots` is set in `config.json`, tell the user Berry looks configured and ask if they want to reconfigure anyway.

## Step 2 — Ask which verifier backend to use

Present these options clearly to the user:

> **Which verifier backend do you want to connect Berry to?**
>
> **1. Local llama.cpp / llama-server** — Any OpenAI-compatible server running locally (e.g. llama-server at `http://127.0.0.1:8080`). No API key needed. Best for privacy and offline use.
>
> **2. OpenAI** — Uses GPT models via the OpenAI API. Requires an OpenAI API key.
>
> **3. Gemini (Google AI Studio)** — Uses Gemini models. Requires a Gemini API key (free tier available at aistudio.google.com).
>
> **4. Other OpenAI-compatible endpoint** — Any server that supports `/v1/completions` with `logprobs`. Includes OpenRouter, vLLM, Ollama, etc.

Wait for the user's answer.

## Step 3 — Collect connection details

Ask only what is needed for their chosen backend:

- **Local llama.cpp**: Ask them to confirm or change the base URL (default: `http://127.0.0.1:8080/v1`). No API key needed — use `"none"`.
- **OpenAI**: Ask for their OpenAI API key (starts with `sk-`). No base URL needed.
- **Gemini**: Ask for their Gemini API key. Explain that Berry will use `GEMINI_API_KEY`.
- **Other**: Ask for the base URL and API key.

## Step 4 — Ask which project directories Berry can read

Berry's `add_file_span` tool reads local files to build evidence spans — but only from directories you explicitly allow. Ask:

> **Which directories should Berry be allowed to read files from?**
>
> Enter full paths (one per line or comma-separated). Examples:
> - `/Users/yourname/repos` — all your projects
> - `/Users/yourname` — your entire home directory

Get their answer. Expand any `~` to the actual home path. Use `echo $HOME` to get the real path if needed.

## Step 5 — Write the configuration files

Create `~/.berry/` if needed. Then write the files based on what the user provided.

**For local llama.cpp or other OpenAI-compatible endpoint:**
```bash
mkdir -p ~/.berry
```
Write `~/.berry/mcp_env.json` with `OPENAI_API_KEY` and `OPENAI_BASE_URL` set to the user's values.

**For OpenAI (no custom base URL):**
Write `~/.berry/mcp_env.json` with only `OPENAI_API_KEY`.

**For Gemini:**
Write `~/.berry/mcp_env.json` with only `GEMINI_API_KEY`.

**`~/.berry/config.json` for all backends** — use the actual expanded paths from Step 4:
```json
{
  "allowed_roots": ["/path/from/step4", "/another/path"],
  "allow_web": false,
  "allow_exec": false,
  "allow_write": false,
  "audit_log_enabled": true
}
```

After writing both files, validate them:
```bash
python3 -c "import json; json.load(open('$HOME/.berry/mcp_env.json')); print('mcp_env.json ✓')"
python3 -c "import json; json.load(open('$HOME/.berry/config.json')); print('config.json ✓')"
```

If either validation fails, show the error and fix the file before continuing.

## Step 6 — Confirm and prompt restart

Tell the user:

> **Berry is configured.** Restart Claude Code to apply the settings — the MCP server reads `~/.berry/mcp_env.json` at startup.
>
> After restarting, Berry will verify claims using **[their chosen backend]**.
>
> To confirm everything works, ask Claude to run the `berry-search-and-learn` skill on any question. You should see it call `start_run`, `add_span`, and `audit_trace_budget` before giving an answer.
