### Supported Platforms

| Web App | Desktop App | VS Code Extension | github.dev |
|:-------:|:-----------:|:-----------------:|:----------:|
| - | ✅ | - | - |

> The Desktop App auto-configures the MCP server for VS Code and Cursor on your machine, so your AI coding assistant can access `.todo` files even outside the app.

### Introduction

Todolist includes a built-in MCP (Model Context Protocol) server that connects your todo files with AI coding assistants like **Claude Code** and **Cursor**. Once set up, your AI assistant can read and understand your `.todo` files directly.

> Let your AI coding assistant work with your todo files!

### Install

Run the following command in your terminal:

```bash
npx skills add https://github.com/aicupa/skills
```

This automatically configures the MCP server for both VS Code and Cursor.

### What Can the AI Do?

After installation, your AI assistant gains the ability to:

- **List** all `.todo` files in your workspace
- **Read** the content of any `.todo` file
- **Understand** the todolist file format and structure

### Usage Examples

Just ask your AI assistant naturally:

- "Show me all my todo files"
- "What tasks are left in my project.todo?"
- "Create a new `.todo` file with a project plan"
- "Build a plugin that shows overdue tasks"

### Supported Assistants

- **Claude Code** — configured via `~/.vscode/mcp.json`
- **Cursor** — configured via `~/.cursor/mcp.json`
- Any other MCP-compatible AI assistant

### Desktop App

If you're using the **Desktop App**, the MCP server is set up automatically on first launch — no extra steps needed.
