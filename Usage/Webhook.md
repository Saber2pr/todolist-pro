### Supported Platforms

| Web App | Desktop App | VS Code Extension | github.dev |
|:-------:|:-----------:|:-----------------:|:----------:|
| - | ✅ | ✅ | ✅ |

### Introduction

Webhook lets your Todolist app automatically send updates to an external service whenever you edit your tasks. This is useful for syncing data to a self-hosted web version or integrating with other tools.

> Auto-sync your changes to external services!

### How to Set Up

1. Open **Settings** in the app.

2. Find the **Webhook** field and enter your target URL. For example:

   - If you deployed the web app via Docker at `http://localhost:3000`, enter:
     ```
     http://localhost:3000/api
     ```

3. Save the settings.

### How It Works

- After you set a Webhook URL, every time you edit, add, or complete a task, the app automatically sends the updated data to that URL.
- No manual action needed — it happens in the background.

### Common Use Case: Docker Web Sync

If you deployed Todolist as a web app using Docker (see [Docker Deploy](DockerDeploy.md)), you can use Webhook to keep the web version in sync with your desktop or VS Code edits:

1. Deploy the web app: `docker run -d -p 3000:3000 saber2pr/todolist-app:master`
2. Set Webhook URL to `http://localhost:3000/api`
3. Edit tasks in VS Code or the Desktop App — the web version updates automatically

### Tips

- The Webhook URL must be a valid HTTP/HTTPS endpoint.
