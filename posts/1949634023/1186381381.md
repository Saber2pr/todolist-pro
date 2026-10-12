> The following features are expected to be updated in the next version

### 2024.3.16

> Here’s a thought about cloud storage

1. The backup function provided by Todolist Tree currently encrypts and saves data in the user directory of your local computer and does not upload cloud backup. Is the cloud backup function a useful feature? Uploading user todolist to the server will involve privacy and security issues. We will seriously consider whether it is necessary to support cloud storage.

If you have any good suggestions, please discuss them on Github ~

[vsc-ext-todolist](https://github.com/Saber2pr/vsc-ext-todolist)

### 2024.3.12

1. It is planned to support theme expansion, background images, label packs, and a theme store for download in the next version.

### 2024.3.11

1. Backup Data: The backup function has been released, you can view the historical version of the current todolist, and it can be accurate to the minute and second. Every edit you make will be recorded as a version, and you can view and safely roll back to the historical version at will. (This feature is only available to pro user)

---

### Plan

- New feature: Support scheduled reminders
- New feature: Support custom plug-in management, define rendering scripts with matching file suffixes
- BUG repair: The new window executes the automated script, and there is a chance of crashing.
   - There is a chance that the clipboard will fail
- New feature: multi-window opening supports opening folders
- New feature: The bottom bar supports expanded customization and can be associated with scripts
   - Settings>Script, change to "Trigger on save"
- New feature: Scheduling Gantt view
   - Month Day
- New feature: formatted paste md
- Optimization: shortcut key optimization
- New feature: tag import and export
- New feature: todo template, new todo and md files support template selection
- Add preview display and hide in the bottom bar of md type
- Version check update function
- animation
- New feature: Cloud storage version
- New feature: Cloud storage backup