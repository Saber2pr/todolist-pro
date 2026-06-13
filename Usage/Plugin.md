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
| `view` | No | Path to frontend view directory (must contain `index.html`) |
| `main` | No | Path to backend service directory (Node.js entry) |
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

The view is an HTML file rendered inside a sandboxed iframe. It communicates with the service via `postMessage`:

```html
<script>
  let callId = 0
  const pending = {}
  let currentFilePath = ''

  // Call a service method
  function callPlugin(method, params) {
    return new Promise((resolve, reject) => {
      const id = ++callId
      pending[id] = { resolve, reject }
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
