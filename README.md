# hermes-plugin-kiwi

Source suppliers and negotiate purchases from Hermes Agent through
[Kiwi](https://github.com/harrylabsj/kiwi)'s buyer MCP server — an open-source
A2A commerce runtime for supplier discovery, RFQ fan-out, negotiation,
non-binding agreements and trade handoff. Kiwi never handles payments and
never places orders.

This is a portable [Agent Plugins v1](https://agent-plugins.org) package. One
install ships:

- `mcp.json` — the Kiwi Buyer MCP server entry (`kb`), registered for every
  new session while the plugin is enabled. No `mcp_servers` edits in
  config.yaml.
- `skills/kiwi-buyer/` — the sourcing workflow skill: when to use which tool,
  the search → RFQ → negotiate → agreement → handoff loop, CommerceIntent
  rules, and the authorization/error-handling contract.

## Install

```bash
hermes plugins install kiwi       # from the Nous plugin catalog (once admitted)
hermes plugins enable kiwi
```

Before catalog admission, install straight from this repo:

```bash
hermes plugins install https://github.com/harrylabsj/hermes-plugin-kiwi
hermes plugins enable kiwi
```

Then start a new Hermes session. The skill is available via `skill_view`
(`skills_list` shows the exact `agent-plugin-kiwi-…` name). Node.js/npm must
be on `PATH` (the server is launched with `npx -y @harrylabsj/kiwi@0.8.0 mcp
serve`). The first launch downloads the pinned package.

Health check without Hermes:

```bash
npx -y @harrylabsj/kiwi@0.8.0 mcp serve <<'EOF'
{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"probe","version":"0"}}}
EOF
```

## What you get

9 tools: `kiwi_search` (supplier discovery), `kiwi_request_quotes` (RFQ),
`kiwi_get_task`, `kiwi_negotiate`, `kiwi_accept_agreement`, `kiwi_get_agreement`,
`kiwi_handoff`, plus the `kiwi_approve` / `kiwi_reject` approval gates.

Hermes exposes them as `mcp__agent_plugin_kiwi_<hash>__kb__<tool>`. The server
key is `kb` rather than `kiwi` so the longest tool name
(`kiwi_accept_agreement`) stays at 58 characters — clear of the 64-character
function-name cap of OpenAI-compatible wire formats.

## How it works

- The buyer discovers merchants through the public Kiwi catalog
  (`https://catalog.kiwi.harrylabsj.com`, built-in default) and then talks to
  merchants directly over A2A. No local marketplace daemon required.
- `kiwi_accept_agreement` and `kiwi_handoff` default to **ask** — the agent
  must show you candidate, terms and amount, and only proceeds after your
  explicit confirmation (`kiwi_approve`). The built-in policy sets payment to
  **never**; Kiwi produces a handoff entry (checkout/PO/contact), it does not
  pay or order.
- State lives in a local SQLite store (default `~/.kiwi/mcp/dsh.sqlite`, HOME
  based). Override with `KIWI_MCP_DB`, `KIWI_PRINCIPAL`, `KIWI_BUYER_AGENT`
  environment variables if needed. No credentials are required at install
  time.

## Version pinning

`mcp.json` pins `@harrylabsj/kiwi@0.8.0`. Bumping the pin is a commit to this
repo followed by a reviewed SHA bump in the
[hermes-agent plugin catalog](https://github.com/NousResearch/hermes-agent/tree/main/plugin-catalog).
`@latest` is deliberately not used.

## Attribution

Kiwi and the Kiwi Buyer MCP server are part of the open-source
[harrylabsj/kiwi](https://github.com/harrylabsj/kiwi) project (Apache-2.0).
This packaging plugin is Apache-2.0 as well.
