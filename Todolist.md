<img height="64px" src="https://cdn.jsdelivr.net/gh/saber2pr/MyWeb@master/resource/image/vsc-todo-v3-logo-title.png" />

---

TodoList TreeView Editor. Check your todo list with a `TreeView` !

> i18n languages supported: English, 中文（简体）, 中文（繁体）, 日本語, 한국어, Deutsch, Français, Español, Italiano, Português (Brasil), Русский, Türkçe

---

## Download

[Vscode Extension](https://marketplace.visualstudio.com/items?itemName=saber2pr.todolist)

[MacOS Desktop App - v0.2.223](https://github.com/Saber2pr/vsc-ext-todolist/releases/download/mac-v0.2.223/Todolist-mac-v0.2.223.tar.gz)

[Windows Desktop App - v0.2.223](hthttps://github.com/Saber2pr/vsc-ext-todolist/releases/download/win-v0.2.223/Todolist-win-v0.2.223.tar.gz)

[Release History](https://github.com/Saber2pr/vsc-ext-todolist/releases?q=v0.2.223&expanded=true)

## Web App

[TodolistTreeView Web](https://app.aicupa.com/)

## Overview

Create and edit `*.todo` file.

---

![loading](https://cdn.jsdelivr.net/gh/saber2pr/MyWeb@master/resource/image/todolist-pro.webp)

Schedule your plans with a calendar view:

![loading](https://cdn.jsdelivr.net/gh/saber2pr/MyWeb@master/resource/image/vsc-ext-todolist-cal-2.webp)

---

## AI-Powered Task Execution

Delegate tasks to any CLI-based AI agent (Claude Code, Cursor, etc.). Click the 🤖 button on a task — the app opens a terminal, sends structured prompts to the agent, and updates task status in real time as the AI completes each item.

- Supports single tasks and task groups (parent with children)
- Includes file links and URLs as context for the AI
- Configurable AI command template in settings

## AI Agent Chat

Built-in AI conversation interface with tool calling support. The agent can directly add, modify, and manage tasks through structured commands.

## MCP Server Integration

Includes a built-in MCP (Model Context Protocol) server for AI coding assistants. One-command setup:

```bash
npx skills add https://github.com/aicupa/skills
```

Provides tools for listing, reading, and generating prompts from `.todo` files — works with Claude Code, Cursor, and other MCP-compatible assistants.

## Plugin System

Full plugin ecosystem with npm marketplace, local installation, and a rich API:

- **Frontend views** rendered in iframe + **backend services** in Node.js
- Context menus, event hooks (`paste`), head views
- Plugin API: file I/O, tree manipulation, clipboard, store, config
- Install from npm (`@aicupa/plugin-*`) or local directory

## Cloud Sync

Sync your todo data across devices with cloud storage:

- **Supabase** backend for data storage
- **Clerk** authentication for secure access
- Automatic sync when enabled

## Backup & Version History

Every edit is recorded as a version — browse, compare (diff viewer), and restore any historical version at will.

## Export Formats

Export your todo tree in multiple formats:

- **Markdown** — hierarchical document with full structure
- **HTML Table** — styled table with colors and tags
- **JSON** — raw tree data
- **Wiki** — article format with titles

## Script & Automation

Create and manage custom scripts with configurable arguments. Execute scripts in the integrated terminal with automatic task binding.

## Search

Full-text search across all `.todo` files in a directory with configurable depth and content preview.

## Desktop App Features

The standalone Electron desktop app includes:

- System tray with quick access
- Global keyboard shortcuts
- Floating quick-add button
- Integrated xterm terminal (multi-tab)
- Native file dialogs and menus
- File watcher for external changes

## Play Mode

Progressive task reveal — display one task at a time with visual progression, ideal for focused execution and presentations.

## Docker Deployment

Deploy as a self-hosted web app:

```bash
docker pull saber2pr/todolist-app:master
docker run -d -p 3000:3000 saber2pr/todolist-app:master
```

## Webhook Integration

Auto-sync todo changes to your deployed web instance or external services via configurable webhook URLs.

## Sharing

Generate share links for read-only access — let others view your todo list without editing permissions.

## Daily Planning

Daily task summary with time-based views:

- Today's tasks and completions
- Hour-by-hour timeline
- Configurable work hours and rest days

## Works on github.dev

Install directly in the browser-based VS Code editor — press `.` on any GitHub repo to start using Todolist in `github.dev`.

---

Shotcuts:

<image width="500px" src="https://cdn.jsdelivr.net/gh/saber2pr/MyWeb@master/resource/image/0603vsc-todolist-p0.png" />
