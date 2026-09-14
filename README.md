# LABHUB-REMOTE

Documentation hub for the **lsuperagent** platform (Enterprise-grade AI development platform).
Browse the deployed documentation at [`docs/index.html`](docs/index.html); see [`version.json`](version.json) for the current release.

---

## MCP Bridge — ChatGPT <-> Superagent

This hub now documents the **MCP bridge** that lets external AI clients such as ChatGPT talk to the owner's Superagent (Base44 personal AI agent, `AGENTS.SDK.MODEL`).

### Overview

The bridge is an MCP server (JSON-RPC 2.0 over Streamable HTTP) deployed as a Base44 backend function. It exposes a single tool:

- **`message_agent`** — sends a message to the Superagent and returns the agent's reply.

```
ChatGPT / MCP client
      |  JSON-RPC (Streamable HTTP)
      v
mcpBridge (Base44 backend function, functions/mcpBridge.ts)
      |  REST + Bearer token
      v
Base44 Superagent Agent API (/api/agents/<agent_id>)
      |
      v
Superagent reply  ->  returned to the client as the tool result
```

### Security

- Requests must present a **bridge token** in one of: `Authorization: Bearer <token>`, `x-api-key: <token>`, or `?key=<token>`. Requests without a valid token get `401`.
- The Superagent API key is stored as an encrypted Base44 secret (`AGENT_API_KEY`); it is never in the source code or in this document.
- The bridge token is defined in the function source (`functions/mcpBridge.ts`) — do not publish it in public docs.

### Endpoint

Once the Superagent app is published, the bridge is reachable at:

```
<app-url>/functions/mcpBridge?key=<bridge-token>
```

**Status: deployed and auth-tested; waiting on app publish for a public URL.**

### Supported MCP methods

| Method | Behavior |
| --- | --- |
| `initialize` | Returns server capabilities (`tools`) and a session id |
| `ping` | Returns `{}` |
| `tools/list` | Returns the `message_agent` tool with its input schema |
| `tools/call` | Executes `message_agent` and returns the agent reply as text |
| `resources/list`, `prompts/list` | Return empty lists |

### Connecting ChatGPT

1. Open ChatGPT **Settings -> Developer mode -> MCPs -> Add server**.
2. Set **Type** to *Streamable HTTP* and paste the bridge endpoint URL above.
3. Save, restart, enable the server, then call `message_agent` from chat.

> Alternative: Base44's native **App MCP** (app dashboard -> MCP -> Set up access -> OAuth -> Publish) provides an official OAuth connection URL that ChatGPT can use directly, without this bridge.

### Notes and caveats

- The Superagent runs a full agent loop; a reply can take from seconds to a few minutes if the agent performs tool work.
- Messages sent through the bridge appear in the owner's Superagent conversation.
- The bridge is a fallback/complement to native App MCP; keep this document in sync with `functions/mcpBridge.ts` when the bridge changes.

---

*This section was added by Thanabat's Superagent on Base44 (`AGENTS.SDK.MODEL`) — the personal AI agent that built and deployed the bridge on 14 September 2026.*
