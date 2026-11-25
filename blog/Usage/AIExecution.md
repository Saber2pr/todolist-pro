### Introduction

AI Execution allows you to delegate todo tasks to an AI agent. Click the AI button on any todo item, and the app will automatically open a terminal, send the task to your AI agent, and the agent will execute the tasks and update their status in real time.

> Let AI work on your tasks while you focus on what matters!

### Setup

1. Open **Settings**, find the **AI Command** field in the Advanced section

2. Enter your AI command template, use `$cmd` as the placeholder for the task prompt. For example:

```
agent --chat "$cmd"
```

The `$cmd` will be automatically replaced with the task description when you trigger AI execution.

### Usage

1. After configuring the AI Command, you will see a **robot icon** (🤖) next to each todo item

2. Click the robot icon on any todo item to start AI execution:

   - **Single task**: If the item has no children, the AI will execute that single task
   - **Task group**: If the item has children, the AI will execute all sub-tasks in order

3. The built-in terminal will automatically open and show the AI agent working on your tasks

### What happens when AI executes

- The todo item is marked as **focus** (highlighted border) to indicate it's being worked on
- The AI agent receives a structured prompt containing:
  - The task list with each item's content and ID
  - The todo file path so the agent can update task status
  - Links and file references attached to each todo (if any)
- As the AI completes each task, it updates the todo file directly
- The app detects file changes and refreshes the UI automatically — you'll see tasks get checked off in real time

### Tips

- **Add links for context**: Attach URLs (`link`) or file paths (`fileLink`) to your todo items. These will be included in the AI prompt, giving the agent more context about what each task involves.

- **Organize with sub-tasks**: Break large tasks into smaller sub-tasks. Click the AI button on the parent item, and the agent will work through each child task sequentially.

- **Terminal stays interactive**: The terminal remains open after AI execution. You can continue using it for other commands or monitor the AI's progress.

- **Works with any AI agent**: The AI Command setting is flexible — it works with any CLI-based AI agent that accepts a text prompt (e.g. Cursor Agent, Claude CLI, custom agents).
