### Plugin System

The desktop app supports a plugin system that allows you to extend functionality via npm packages or local directories. Plugins can provide both a frontend view (rendered in an iframe) and a backend service (Node.js code with access to internal APIs).

> Plugin system is available in the **Desktop App** only.

## Plugin Structure

A plugin is a directory with a `package.json`:

```
my-plugin/
  package.json
  view/
    index.html
  service/
    index.js
```

### package.json

```json
{
  "name": "@aicupa/plugin-my-plugin",
  "version": "1.0.0",
  "description": "My plugin description",
  "view": "./view",
  "main": "./service",
  "viewSize": {
    "width": 800,
    "height": 600
  },
  "pluginContributes": {
    "contextMenus": [
      {
        "title": "My Action",
        "title_zh": "我的操作",
        "command": "myServiceMethod"
      }
    ],
    "events": {
      "paste": {
        "check": "hasPendingData",
        "handler": "onPaste"
      }
    }
  },
  "dependencies": {
    "@aicupa/api": "^1.0.1"
  }
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Plugin name, recommended prefix `@aicupa/plugin-` |
| `version` | Yes | Semver version |
| `description` | No | Short description |
| `view` | No | Path to frontend view directory (must contain `index.html`), or a URL (`https://...`) to load directly in an iframe |
| `main` | No | Path to backend service directory (Node.js entry) |
| `viewSize` | No | Custom modal size for plugin view `{ "width": 800, "height": 600 }`, default 600×400 |
| `pluginContributes` | No | Declare UI extensions and event handlers (see below) |

### pluginContributes

The `pluginContributes` field lets a plugin extend the app UI and hook into app events declaratively.

#### contextMenus

Register items in the tree node right-click context menu. Each item maps to a service method.

```json
"contextMenus": [
  {
    "title": "Cut",
    "title_zh": "剪切",
    "command": "cutNode"
  }
]
```

| Field | Required | Description |
|-------|----------|-------------|
| `title` | Yes | Menu item label (English) |
| `title_zh` | No | Menu item label (Chinese), falls back to `title` |
| `command` | Yes | Service method name to call. Receives `{ node, filePath }` |

When clicked, the app calls the plugin's service method with:
```js
{ node: { key, children, todo: { id, content, done, ... } }, filePath: "/path/to/file" }
```

#### events

Hook into app-level events. Currently supported:

**`paste`** — Intercept the paste action on tree nodes. When a user clicks "Paste", the app checks all plugins with a `paste` event before performing the default clipboard paste.

```json
"events": {
  "paste": {
    "check": "getCutBuffer",
    "handler": "pasteNode"
  }
}
```

| Field | Description |
|-------|-------------|
| `check` | Service method called to check if this plugin has pending data. Must return `{ ok: true, result: { node: ... } }` if active, `{ ok: true, result: { node: null } }` if not |
| `handler` | Service method called to perform the paste. Receives `{ targetNode, filePath }` |

If `check` returns a non-null node, the app skips the default clipboard paste and calls `handler` instead. After a successful handler call, the tree is reloaded automatically.

#### views

Register plugin views into specific areas of the app UI. The view HTML is loaded from the plugin's `view` directory and rendered inline as an iframe.

```json
"views": {
  "head": true
}
```

| Field | Description |
|-------|-------------|
| `head` | Render the plugin view at the top of the todolist, above the progress bar |

**Head view** receives tree data automatically whenever the todolist updates. The host sends:

```json
{ "type": "plugin-tree-update", "tree": [...] }
```

The view can also request the current tree data at any time:

```json
// View -> Host
{ "type": "plugin-request-tree" }

// Host -> View (response)
{ "type": "plugin-tree-update", "tree": [...] }
```

The `tree` is the full todolist tree in plain format, with each node containing `{ key, children, todo: { id, content, done, date, ... } }`.

Head views also support the standard `plugin-call` / `plugin-result` protocol for calling service methods, and receive `plugin-init` on load.

## Service

Add `@aicupa/api` as a dependency to get full type hints. Use `createPlugin` to define the service:

```js
const { createPlugin } = require('@aicupa/api')

module.exports = createPlugin((api) => {
  return {
    async myMethod(params) {
      // api has full type hints (PluginApi)
      const tree = await api.getTree(params.filePath)
      // ...
      return { ok: true }
    },
  }
})
```

The `createPlugin` wrapper is optional (it's an identity function at runtime), but it provides TypeScript/IDE type inference for the `api` parameter.

You can also use the plain export format:

```js
module.exports = function (api) {
  return {
    async myMethod(params) {
      const tree = await api.getTree(params.filePath)
      return { ok: true }
    },
  }
}
```

### API Reference

| API | Description |
|-----|-------------|
| `api.reload(filePath)` | Refresh the todolist view (without full page reload) |
| `api.getTree(filePath)` | Get the todolist JSON tree data for the given file |
| `api.store(key, value, filePath)` | Save data to the todolist store |
| `api.relaunch()` | Restart the entire Electron app |
| `api.setTitle(title, filePath)` | Set the window title |
| `api.getConfig()` | Get app configuration |
| `api.readFile(path)` | Read a file (returns string) |
| `api.writeFile(path, content)` | Write content to a file |
| `api.readdir(dir)` | List files in a directory |
| `api.mkdir(dir)` | Create a directory (recursive) |
| `api.remove(path)` | Remove a file or directory |
| `api.pathJoin(...args)` | Join path segments |
| `api.mapTree(tree, fn)` | Recursively map over a tree structure |
| `api.getArray(val)` | Safely convert a value to an array |
| `api.base64(str)` | Encode a string to base64 |
| `api.isWindows` | `true` if running on Windows |

## View

The `view` field supports two modes:

- **Local directory** (e.g. `"./view"`) — loads `view/index.html` inside a sandboxed `srcDoc` iframe. Supports `postMessage` communication with the service.
- **URL** (e.g. `"https://example.com/app/"`) — loads the URL directly in an iframe with `src`. No sandboxing or `postMessage` bridge — useful for embedding external web apps.

```json
// Local view
"view": "./view"

// URL view
"view": "https://saber2pr.top/editor/"
```

### Local View

The local view is an HTML file rendered inside a sandboxed iframe. It communicates with the service via `postMessage`:

```html
<script>
  let callId = 0
  const pending = {}
  let currentFilePath = ''

  // Call a service method (with timeout to prevent hanging)
  function callPlugin(method, params) {
    return new Promise((resolve, reject) => {
      const id = ++callId
      const timer = setTimeout(() => {
        delete pending[id]
        reject(new Error('timeout'))
      }, 2000)
      pending[id] = {
        resolve: (v) => { clearTimeout(timer); resolve(v) },
        reject: (e) => { clearTimeout(timer); reject(e) },
      }
      window.parent.postMessage(
        { type: 'plugin-call', id, method, params },
        '*'
      )
    })
  }

  // Handle responses and init
  window.addEventListener('message', (event) => {
    const msg = event.data

    // Receive filePath from host on init
    if (msg?.type === 'plugin-init') {
      currentFilePath = msg.filePath || ''
      return
    }

    // Handle service call results
    if (msg?.type === 'plugin-result' && pending[msg.id]) {
      const { resolve, reject } = pending[msg.id]
      delete pending[msg.id]
      if (msg.error) {
        reject(new Error(msg.error))
      } else {
        resolve(msg.result)
      }
    }
  })

  // Example: call service method
  async function doSomething() {
    const result = await callPlugin('myMethod', {
      filePath: currentFilePath,
    })
    console.log(result)
  }
</script>
```

### Pitfalls

#### callPlugin must have a timeout

The `callPlugin` function uses `postMessage` to communicate with the host. If the host or service is not ready when the message is sent (e.g., `plugin-init` hasn't fired yet, or the service hasn't initialized), the `plugin-result` response will never arrive — causing the Promise to hang forever. This silently blocks all downstream logic (data loading, saving, etc.).

**Always add a timeout** (e.g., 2 seconds) to `callPlugin` so it rejects on time and fallback logic can run.

#### Use localStorage as a fallback for view-side persistence

Because `callPlugin` can fail silently (timeout, service not ready), don't rely solely on the service for persisting view-side data like user input. Write to `localStorage` immediately as a synchronous backup, and attempt the service call in parallel:

```js
function saveData(val) {
  localStorage.setItem('my_key', val)               // instant, reliable
  callPlugin('save', { value: val }).catch(() => {}) // async, best-effort
}

async function loadData() {
  try {
    const res = await callPlugin('load', {})         // try service first
    if (res?.result?.value) return res.result.value
  } catch {}
  return localStorage.getItem('my_key')              // fallback
}
```

### Message Protocol

**View -> Host** (call service):
```json
{ "type": "plugin-call", "id": 1, "method": "myMethod", "params": {} }
```

**Host -> View** (init with filePath):
```json
{ "type": "plugin-init", "filePath": "/path/to/file" }
```

**Host -> View** (service result):
```json
{ "type": "plugin-result", "id": 1, "result": {} }
```

## Installation

### From npm

Plugins published to npm with the `@aicupa/plugin-` prefix can be searched and installed from the Plugin Marketplace (click the plugin icon in the bottom toolbar).

- Searching a full package name (e.g. `@aicupa/plugin-overdue-tasks`) uses a direct registry lookup for faster results
- Installation downloads the tarball directly from `registry.npmjs.org` (bypasses local npm config / private registries)
- Before installing, the app validates that the package has a `view` field in its `package.json`. Packages without it are rejected as invalid plugins

### From local directory

1. Click the plugin icon in the bottom toolbar
2. Select "Plugin Market" from the dropdown
3. Click "Install from local" in the bottom bar
4. Select the plugin directory

## Storage

- Plugins are installed to `~/.todoListNative/plugins/`
- Plugin registry is stored at `~/.todoListNative/plugins.json`
- The `@aicupa/api` package is automatically provisioned at `~/.todoListNative/plugins/node_modules/@aicupa/api/` so all plugins can `require` it without bundling

## AI Skill (for Claude Code / Cursor / etc.)

The todolist-app repo includes a built-in **skill** that packages all todolist knowledge (file format, plugin system, plugin API, view protocol) for AI coding assistants. Once installed, your AI assistant understands how to work with `.todo` files and build plugins.

### Install

```bash
npx skills add https://github.com/aicupa/skills
```

### What's Included

The skill provides:

- **`.todo` file format** — complete structure reference (`todotree` wrapper, tree nodes, todo items, tags, timelines)
- **Plugin system** — directory structure, `package.json` fields, `pluginContributes` (contextMenus, events, views.head)
- **Plugin API** — all `api.*` methods (`getTree`, `store`, `reload`, `clipboard`, `readFile`, etc.)
- **View protocol** — `postMessage` message types (`plugin-call`, `plugin-result`, `plugin-init`, `plugin-tree-update`, `plugin-request-tree`)
- **Best practices** — timeout handling, localStorage fallback, theme adaptation, auto-height for head views

### Usage

After installing the skill, just ask your AI assistant to work with todolist files or plugins — it will automatically use the skill's knowledge. For example:

- "Create a new `.todo` file with a project structure"
- "Build a plugin that shows overdue tasks"
- "Add a context menu item to my plugin"
- "Create a head view that displays task statistics"

## Examples

Official plugin examples are available at [github.com/aicupa](https://github.com/aicupa) for reference.
