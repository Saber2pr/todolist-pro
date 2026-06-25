### Supported Platforms

| Web App | Desktop App | VS Code Extension | github.dev |
|:-------:|:-----------:|:-----------------:|:----------:|
| - | ✅ | ✅ | ✅ |

### Introduction

The Script system lets you create and run custom commands or scripts directly from within the app. You can set up frequently used commands, bind them to tasks, and execute them in the built-in terminal.

> Automate repetitive tasks with custom scripts!

### How to Use

#### Creating a Script

1. Open the **Scripts** page from the toolbar.

2. Click **Add** to create a new script.

3. Fill in the form:
   - **Type**: Choose **File** (run a script file) or **CLI** (run a command)
   - **Path / Command**: The file path to your script, or the command to run
   - **Description**: A short note to remind you what this script does
   - **Auto Close**: Toggle on if you want the terminal to close after the script finishes

4. Click **Save**.

#### Running a Script

1. In the Scripts page, find the script you want to run.

2. Click the **Run** button — the built-in terminal will open and execute your script.

3. You can see the output in real time in the terminal.

### Examples

| Type | Path / Command | Description |
|------|---------------|-------------|
| CLI | `npm run build` | Build the project |
| CLI | `git status` | Check git status |
| File | `./scripts/deploy.sh` | Deploy to production |
| CLI | `python report.py` | Generate weekly report |

### Tips

- Scripts are saved to your workspace, so they persist between sessions.
- Use **CLI** type for quick one-liner commands, and **File** type for complex multi-step scripts.
- The terminal stays open after execution so you can review the output or run follow-up commands.
