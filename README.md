# Exomemory plugin

Exomemory lets ChatGPT, Claude, Codex and Claude Code draw on your life context
and history: past messages, ideas and decisions from your WhatsApp, Messenger,
iMessage and Telegram chats, your Obsidian, Notion and Apple Notes, and your
voice memos. With that background your assistant can brief you before a
meeting, build a timeline of how something came about, or find who recommended
what and when, with dated excerpts and links to the original records. Your
archive runs on your own computer.

Things to ask:

- "I'm planning the Porto trip with Maya. Brief me on what we've agreed so far."
- "Build a timeline of the key decisions on Project Kestrel, with links to the original notes."
- "Find the voice memo where I explained why I changed my sourdough recipe."

This repository is generated from the Exomem source repository
(`plugins/` there, `pnpm plugins:package`); edit it there, not here.

## What the plugin contains

| Path | Purpose |
| --- | --- |
| `skills/exomemory/` | How to search and cite the archive well |
| `skills/setup/` | Guided setup: install Exomem, request exports, import them, connect assistants |
| `.claude-plugin/plugin.json`, `.mcp.json` | Claude plugin manifest and its MCP server entry |
| `.claude-plugin/marketplace.json` | Lets Claude Code install the plugin straight from this repository |
| `plugin.json`, `mcp.json` | OpenAI (ChatGPT and Codex) plugin manifest and its MCP server entry |
| `assets/icon.png` | Icon |

## What it runs, sends and fetches

- **One remote endpoint**: `https://mcp.exomem.io/mcp` is the only server the
  plugin talks to. Gevolut Oy runs it on a server in Helsinki, Finland; Cloudflare
  carries the traffic. The plugin starts no local processes, has no hooks and
  contains no scripts or binaries.
- **Without signing in**, the tools `about` and `capabilities` and the tool list
  work, answered from a demo workspace with synthetic memories.
- **The memory tools** (`search`, `get`, `expand`, `list_collections`,
  `list_records`, `resolve_person`) ask you to sign in at exomem.io (email link,
  OAuth). exomem.io then forwards each request to the Exomem running on your own
  computer over its outbound link, and the answer back. It does not store
  questions or answers; it keeps your email address, your connections, your
  paired device and 30 days of outcome logs (tool name, status, duration).
  Without a paired computer you get the demo workspace.
- **`log_feedback`** sends a one-line note about the Exomem service itself to
  exomem.io, which keeps it to improve Exomem (detached from your account if you
  delete it). A note must be redacted or contain only limited details you have
  agreed to share.
- **The setup skill** runs commands on your computer only in assistants that can
  (Claude Code, Codex), and only after you approve each one: it clones the
  Exomem repository from GitHub, installs its dependencies, starts a local
  database and embedding model, and imports the exports you choose. It never
  asks for platform passwords and never uploads your exports.

## Install

- Claude Code: `/plugin install exomemory --marketplace WilliamNurmi/exomemory-plugin`
  (this repository is its own one-plugin marketplace), or install it from the
  Claude directory once listed.
- ChatGPT and Codex: install Exomemory from the plugin directory once listed.
- Any MCP client: connect `https://mcp.exomem.io/mcp` directly.

Set up your own Exomem: https://exomem.io/setup · Privacy: https://exomem.io/privacy ·
Terms: https://exomem.io/terms · Support: https://exomem.io/support (hello@exomem.io)
