---
name: setup
description: Set up a self-hosted Exomem on the user's computer, import exports of their chats, notes and voice memos, and connect it to ChatGPT, Claude, Claude Code or Codex. Use when the user wants to install Exomem, add a new source to it, or connect it to an assistant.
---

# Set up Exomem

Exomem runs on the user's own computer. Their exports and the database stay
there; exomem.io only relays questions from their assistants to it.

## Where you are running

- **With a terminal on the user's computer** (Claude Code, Codex, Claude desktop
  with local tools): follow the steps below. Show each command and what it does
  before running it, and run it only with the user's approval.
- **In a chat without access to their computer** (ChatGPT, claude.ai, phones):
  you cannot run anything there. Explain the steps briefly, link
  https://exomem.io/setup, and offer to continue in Claude Code or Codex.

## Ground rules

- Re-running is safe. Before each step, check whether it is already done and
  skip it if so; this lets the user stop and resume, for example while an export
  is being prepared.
- Ask before installing software. Prefer the user's existing package manager.
- Never ask for, type or store passwords for Facebook, Google or any other
  platform. The user signs in to those sites themselves.
- Exports contain other people's messages. Keep them on this computer; never
  upload them anywhere.

## 1. Preflight

Check, and report in one short table:

| Need | Check | If missing |
| --- | --- | --- |
| macOS or Linux | `uname -sm` | Windows: use WSL2 (not yet tested) |
| Node 22 | `node --version` | nodejs.org installer or nvm |
| pnpm | `corepack pnpm --version` | `corepack enable` |
| git | `git --version` | Xcode Command Line Tools / the distro package |
| Docker with Compose | `docker compose version` | Docker Desktop (macOS) or Docker Engine (Linux) |
| uv (embeddings) | `uv --version` | the official uv installer |
| Disk | `df -h ~` | at least 10 GB free (embedding model and database) |
| Acceleration | Apple Silicon (`arm64`) or `nvidia-smi` | CPU works, more slowly |

## 2. Install

Done already if an Exomem checkout exists and `pnpm project:doctor` passes there.

```sh
git clone https://github.com/WilliamNurmi/exomem.git ~/Exomem
cd ~/Exomem
pnpm project:setup      # .env, dependencies, local Postgres in Docker, migrations
pnpm project:doctor
```

Embeddings (needed for meaning-based search):

- Apple Silicon or an NVIDIA GPU: `pnpm embed:service:install`, then keep
  `pnpm embed:service` running. The first start downloads the model (about
  1–2 GB).
- Otherwise: set `EXOMEM_STACK_EMBEDDINGS_CPU=true` in `.env` and run
  `pnpm stack:up`.

Finish with `pnpm project:smoke`.

## 3. Choose sources

Run `pnpm sources:status --json`. Offer the sources with `status: "implemented"`
first, saying for each whether an export is already on disk (`exportPresent`)
and how many records are imported (`ingestedRecordCount`). Then list the `export-only` sources honestly: Exomem
cannot import them yet, but the user can request the export now, since some take
days to arrive. Ask which ones to set up.

## 4. Get the exports

For each chosen source, run `pnpm sources:guide <key>` and walk the user through
it: what to request, where to click, how long it takes, how it arrives and
where to put the files.

When the export happens in a website or arrives by email, you may offer help if
the user has a browser or email tool enabled in this assistant: say exactly what
you would do (for example "open the export page" or "find the download email")
and do it only after the user says yes. Otherwise give the steps and wait. The
user signs in to the platform themselves.

Exports can take hours or days. Tell the user they can come back and run this
setup again; it resumes where it stopped.

## 5. Import and check

Run the source's import command from `pnpm sources:status --json`
(`ingestCommand`, for example `pnpm ingest:obsidian`; `requirements` names
anything it needs first). Then show that it works
with a question the user would recognise:

```sh
pnpm query:search --query="<something the user remembers>"
pnpm sources:status
```

Then say once: everything imported is reachable from the user's connected
assistants through exomem.io, and `pnpm exposure:set <source> --local-only` keeps
a source on this computer only. Change the exposure settings only if the user
asks.

## 6. Connect assistants

On this computer (direct, no exomem.io involved):

- Claude Code: `claude mcp add exomem -- pnpm --dir ~/Exomem mcp:stdio`
- Codex: add to `~/.codex/config.toml`:

  ```toml
  [mcp_servers.exomem]
  command = "pnpm"
  args = ["--dir", "/Users/<you>/Exomem", "mcp:stdio"]
  ```

From anywhere (ChatGPT, claude.ai, phones), through exomem.io:

1. `pnpm link:pair` prints a link and a code. The user opens the link, signs in
   with their email (a sign-in link arrives by email) and confirms the code.
2. Keep the link running: `pnpm link:install-service --load` installs a login
   service (ask first), or `pnpm link:start` runs it in this terminal.
3. In ChatGPT or Claude, connect Exomem (the Exomem app or plugin, or the
   connector URL `https://mcp.exomem.io/mcp`) and ask a question about one of
   the imported sources. Answers no longer carry the demo notice.

`pnpm link:status` shows whether this computer is paired and connected.

## When something fails

Run `pnpm project:doctor` first and act on what it reports. For connection
problems, `pnpm link:status` explains the state. Exomem's documentation is in
the checkout (`README.md`, `docs/setup.md`, `docs/link.md`).
