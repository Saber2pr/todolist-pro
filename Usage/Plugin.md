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
  "main": "./service"
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Plugin name, recommended prefix `@aicupa/plugin-` |
| `version` | Yes | Semver version |
| `description` | No | Short description |
| `view` | No | Path to frontend view directory (must contain `index.html`) |
| `main` | No | Path to backend service directory (Node.js entry) |

## Service

The service entry file exports a function that receives an `api` object and returns an object of methods:

```js
module.exports = function (api) {
  return {
    async myMethod(params) {
      // use api to interact with the app
      const tree = await api.getTree(params.filePath)
      // ...
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

### From local directory

1. Click the plugin icon in the bottom toolbar
2. Select "Plugin Market" from the dropdown
3. Click "Install from local" in the bottom bar
4. Select the plugin directory

## Storage

- Plugins are installed to `~/.todoListNative/plugins/`
- Plugin registry is stored at `~/.todoListNative/plugins.json`
