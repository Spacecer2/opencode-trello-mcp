# opencode Trello MCP

Drop-in Trello MCP server config for [opencode](https://opencode.ai).
Add Trello board management as an MCP tool to any opencode agent session.

## What it does

Adds the `trello` MCP server (backed by [`mcp-trello`](https://www.npmjs.com/package/mcp-trello))
with these tools:

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

## 1 — Get your Trello credentials

### API Key

1. Go to https://trello.com/app-key
2. Copy the **API key** shown at the top.

### Token

1. On the same page click **Generate a Token** (or visit the link below).
2. Authorize the token. Select the boards you want the bot to see.

**One-click token link** (replace `YOUR_API_KEY` with the key from step 1):

```
https://trello.com/1/authorize?expiration=never&scope=read,write&response_type=token&key=YOUR_API_KEY
```

Copy the 64-character hex string the page returns.

### Board ID (optional, but recommended)

Some tools (`getLists`, `addList`, `getRecentActivity`) require a board ID.

The **short link** in your Trello URL is **not** the board ID.
To find the real 24-character ID:

1. Configure the server with just your API key and token (leave `TRELLO_BOARD_ID` unset).
2. Ask the agent to run `getMyBoards` — the response includes the full `id` for each board.
3. Copy the `id` (24 hex characters, e.g. `69999543c91c992d05b9f352`) into your `.env`.

---

## 2 — Export your environment variables

Add these to your shell profile (`~/.zshrc`, `~/.bashrc`, `~/.profile`)
or an `.env` file that is sourced before launching opencode.

**Never commit these values to a repository.**

```bash
export TRELLO_API_KEY="your-api-key-here"
export TRELLO_TOKEN="your-token-here"
export TRELLO_BOARD_ID="your-board-id"   # optional initially
```

---

## 3 — Add the MCP server to opencode

Open your opencode config:

- Global: `~/.config/opencode/opencode.jsonc`
- Or project-local: `.opencode/opencode.jsonc`

Add the following inside the `"mcp"` block:

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

## .env.example

Copy this to `.env` and fill in your values (`.env` should be in `.gitignore`):

```
TRELLO_API_KEY=
TRELLO_TOKEN=
TRELLO_BOARD_ID=
```

---

## Notes

- The token type must be a **member token** (not an API key-only token).
- Tokens created with `expiration=never` do not expire.
- The server runs via `npx` so Node.js ≥ 18 must be installed.
- Rate limiting is handled automatically (Trello: 300 req/10s per API key, 100 req/10s per token).
