# opencode Trello MCP — Legatus project setup

Drop-in Trello MCP server config for [opencode](https://opencode.ai),
tuned for the **Legatus** project. Add the Legatus Trello board as an MCP tool
to any opencode agent session so an agent can read, create, and move project
work directly.

## Project: Legatus

- **Repo:** https://github.com/eccemono/legatus (private — the agent needs your GitHub auth to read it)
- **Board:** https://trello.com/b/GYgZAlNN/legatus
- **Board ID:** `6aa68c6f28dd8180c8140fc5` (already set in `TRELLO_BOARD_ID`)
- **What it is:** a Discord-native bot with a local-first LLM agent core — verification & moderation,
  HEXACO psychometrics, LLM cognition with per-participant continuity and provenance-marked answers,
  GPU-side inference broker, loopback web tooling. Node 20 + TypeScript, CI on every push.

### The pipeline (list order on the board)

| List | Meaning |
|---|---|
| **Backlog** | Feature ideas, not yet ready to start |
| **To Do** | Agreed, ready to pick up |
| **In Progress** | Being worked on |
| **Done** | Shipped |

The board also uses labels: `bug`, `security`, `enhancement`, `performance`, `docs`, `shipped`.
Feature ideas land in **Backlog**; bugs and ready work go to **To Do**; move to **In Progress**
when you start and **Done** when merged.

---

## 1 — Get your Trello credentials

### API Key

1. Go to https://trello.com/app-key
2. Copy the **API key** shown at the top.

### Token

1. On the same page click **Generate a Token** (or visit the link below).
2. Authorize the token.
3. Copy the 64-character hex string the page returns.

**One-click token link** (replace `YOUR_API_KEY` with the key from step 1):

```
https://trello.com/1/authorize?expiration=never&scope=read,write&response_type=token&key=YOUR_API_KEY
```

### Board ID

For Legatus this is already known: `6aa68c6f28dd8180c8140fc5`
(https://trello.com/b/GYgZAlNN/legatus). The short link is **not** the ID.
If you ever need to re-discover it, ask the agent to run `getMyBoards` and copy
the full 24-character `id` field from the Legatus row.

---

## 2 — Export your environment variables

Add these to your shell profile (`~/.zshrc`, `~/.bashrc`, `~/.profile`)
or an `.env` file that is sourced before launching opencode.

**Never commit these values to a repository.**

```bash
export TRELLO_API_KEY="your-api-key-here"
export TRELLO_TOKEN="your-token-here"
export TRELLO_BOARD_ID="6aa68c6f28dd8180c8140fc5"
```

---

## 3 — Add the MCP server to opencode

Open your opencode config:

- Global: `~/.config/opencode/opencode.jsonc`
- Or project-local: `.opencode/opencode.jsonc`

Add the following inside the `"mcp"` block (already done on the owner machine;
this is for anyone else picking up the project):

```jsonc
"trello": {
  "type": "local",
  "command": ["npx", "-y", "mcp-trello"],
  "enabled": true,
  "environment": {
    "TRELLO_API_KEY": "{env:TRELLO_API_KEY}",
    "TRELLO_TOKEN": "{env:TRELLO_TOKEN}",
    "TRELLO_BOARD_ID": "{env:TRELLO_BOARD_ID}"
  }
}
```

The `{env:VAR}` syntax tells opencode to read the value from your process
environment at launch, so the actual keys never appear in the config file.

---

## 4 — Restart opencode

Kill and restart your opencode session.
Verify the server is loaded:

```bash
opencode mcp list
```

You should see:

```
●  ✓ trello connected
```

---

## 5 — First-connect checklist for the agent

When an agent session starts and Trello is connected, have it run this checklist
before touching the board:

1. `getMyBoards` → confirm the Legatus board is visible (`GET` the id).
2. `getLists` (board scoped by `TRELLO_BOARD_ID`) → confirm lists are
   `Backlog / To Do / In Progress / Done`, in that order.
3. `getCardsByList` on **Backlog** and **To Do** → understand what is pending.
4. Read card descriptions for feature ideas (they carry the spec/context of the request).
5. Only then: add cards to **Backlog** for new ideas, move to **To Do** when agreed,
   **In Progress** while working, **Done** when merged.
6. Never invent board IDs — always use the board from `TRELLO_BOARD_ID` or `getMyBoards`.

---

## What the tools can do

| Tool | Description |
|---|---|
| `getMyBoards` | List all boards the authenticated user has access to |
| `getMyCards` | Cards assigned to the authenticated user |
| `getCardsByList` | All cards in a specific list |
| `addCard` | Create a new card |
| `updateCard` | Update name, description, due dates, labels, position |
| `moveCard` | Move a card to another list or board |
| `archiveCard` | Archive a card |
| `changeCardMembers` | Add/remove members from a card |
| `getLists` | Retrieve all lists on a board |
| `addList` | Add a new list |
| `archiveList` | Archive a list |
| `getRecentActivity` | Board activity feed |

---

## .env.example

Copy this to `.env` and fill in your values (`.env` should be in `.gitignore`):

```
TRELLO_API_KEY=
TRELLO_TOKEN=
TRELLO_BOARD_ID=6aa68c6f28dd8180c8140fc5
```

---

## Notes

- The token must be a **member token** (not an API key-only token).
- Tokens created with `expiration=never` do not expire.
- The server runs via `npx` so Node.js ≥ 18 must be installed.
- Rate limiting is handled automatically (Trello: 300 req/10s per API key, 100 req/10s per token).