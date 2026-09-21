### Introduction

The Docker-deployed web board is a **read-only board maintained by an AI agent** — perfect for weekly plans, project boards, and progress dashboards that your agent keeps up to date while you just check them in a browser.

> Your AI agent manages the board; you just watch it.

The board exposes a plain HTTP JSON API, so an agent (Claude Code, Codex, or any script) can read and write the todo tree directly — no app, no manual editing.

### Step 1: Deploy the Board (Docker)

```bash
docker pull saber2pr/todolist-app:master
docker run -d -p 3000:3000 saber2pr/todolist-app:master
```

Then confirm the board is up at `http://<board-host>:3000/`. See [DockerDeploy](/posts/Usage/DockerDeploy) for details.

### Step 2: Install the Skill

Give your AI agent the Todolist knowledge so it knows the `.todo` format, the board API, and the pitfalls (second-vs-millisecond timestamps, full-snapshot writes, `eventId` ordering):

```bash
npx skills add https://github.com/aicupa/skills
```

This installs the `todolist-knowledge` skill, which covers the file format, the board's `/api` events (`Store` / `GetStore` / `saveConfig`), calendar scheduling, monthly milestone tags, and weekly board maintenance workflows.

### Step 3: Tell Your AI the Board Address

In your agent chat, say something like:

> My todo web board is at `http://board.local:3000`, uid `data`, file `2026/W38.todo`. Keep it in sync with my weekly plan.

From then on the agent can manage the board end to end:

- **Read** the current tree: POST `{"service":"GetStore","params":{"key":"todotree","path":"<file>"}}`
- **Write** the whole store back: POST a `Store` event with the full todotree snapshot and `eventId` set to the store's `version`
- **Create a new period file** (e.g. next week's board): `Store` to a new `file` path — the board auto-creates it and switches `currentKey` to it

The skill's `webhook-sync` reference has the exact request shapes and a round-trip recipe.

### Tips

- Keep the `.todo` files in a git repo when possible — you get history, and human + agent can collaborate on the same board (pull before editing, push after).
- The bottom bar of a Docker web board shows a **For AI Agent** link pointing to this guide.
