# Berry Setup

Install the Berry Python CLI so the MCP server can start.

Run the following commands to complete Berry setup after installing this plugin:

```bash
# 1. Install the Berry CLI (requires Python 3.10+ and pipx)
pipx install -e "$(claude plugin path berry)"

# 2. Configure your verifier backend
berry setup

# 3. Verify the MCP server starts
echo '{"jsonrpc":"2.0","method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"0"}},"id":1}' \
  | berry mcp --server classic
```

**Verifier backends supported:** OpenAI, Gemini, Vertex AI, or any OpenAI-compatible endpoint (e.g. local llama.cpp server at `http://127.0.0.1:8080/v1`).

Note: Anthropic's API does not support `logprobs` and cannot be used as the verifier backend.
