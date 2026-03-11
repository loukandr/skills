# NotePlan Plugin Development

Complete guide to building NotePlan plugins: architecture, API reference, HTMLView patterns, plugin.json schema, and publishing.

## Table of Contents

- [Quick Orientation](#quick-orientation)
- [Plugin Types](#plugin-types)
- [Core API Overview](#core-api-overview)
- [Development Workflow](#development-workflow)
- [Essential Patterns](#essential-patterns)
- [Plugin Hooks and Triggers](#plugin-hooks-and-triggers)
- [Sidebar Pinning](#sidebar-pinning)
- [JavaScript Engine Constraints](#javascript-engine-constraints)
- [Common Pitfalls](#common-pitfalls)
- [plugin.json Schema](#pluginjson-schema)
- [API Reference: NotePlan](#api-reference-noteplan)
- [API Reference: Editor](#api-reference-editor)
- [API Reference: DataStore](#api-reference-datastore)
- [API Reference: CommandBar](#api-reference-commandbar)
- [API Reference: Calendar](#api-reference-calendar)
- [API Reference: Clipboard](#api-reference-clipboard)
- [API Reference: HTMLView](#api-reference-htmlview)
- [API Reference: Types](#api-reference-types)
- [API Reference: fetch()](#api-reference-fetch)
- [HTMLView Architecture](#htmlview-architecture)
- [HTMLView Communication Patterns](#htmlview-communication-patterns)
- [HTMLView Theme Integration](#htmlview-theme-integration)
- [HTMLView Auto-Refresh Pattern](#htmlview-auto-refresh-pattern)
- [HTMLView Production Checklist](#htmlview-production-checklist)
- [File Organization](#file-organization)
- [Publishing](#publishing)
- [Requirements for Acceptance](#requirements-for-acceptance)

---

## Quick Orientation

A plugin is a folder containing at minimum two files:

```
YourId.PluginName/
├── plugin.json          # Metadata, commands, settings schema
└── script.js            # JavaScript executed by NotePlan's engine
```

The folder lives inside NotePlan's Plugins directory (Preferences > Plugins > "Open Plugin Folder"). The folder name must match `plugin.id` in plugin.json.

## Plugin Types

### 1. Command-Only Plugins
Run logic when the user invokes a /command. Interact through CommandBar prompts, modify notes via Editor/DataStore, and return.

### 2. HTMLView Plugins
Open a full HTML/CSS/JS interface in a window, sheet, or the main sidebar area. The HTML communicates bidirectionally with the plugin's backend JavaScript.

## Core API Overview

NotePlan injects global objects into the plugin's JavaScript runtime:

| Object | Purpose |
|---|---|
| `NotePlan` | Environment info (theme, platform, version) |
| `Editor` | Current note: read/write content, paragraphs, selection |
| `DataStore` | Query/create/move notes, access settings, invoke other plugins |
| `CommandBar` | Prompt user input: text, options list, date picker |
| `Calendar` | Create events/reminders, parse date strings |
| `Clipboard` | Read/write system clipboard |
| `HTMLView` | Open HTML windows, run JavaScript in WebView |

## Development Workflow

### 1. Create Files
Start with `plugin.json` and `script.js`. Define at least one command in `plugin.commands` that maps a `/command` name to a `jsFunction`.

### 2. Write the Function
Every command's `jsFunction` must be a top-level function in script.js. Use `async` if calling any Promise-returning API.

```javascript
async function myCommand() {
  try {
    const input = await CommandBar.textPrompt('Title', 'Enter something:')
    if (!input) return
    Editor.insertTextAtCursor(input)
  } catch (error) {
    console.log('Error: ' + String(error))
  }
}
```

### 3. Install and Test
Drop the folder into the Plugins directory. Open Command Bar (CMD+J), type `/` and your command name. Use Help > Plugin Console for `console.log` output.

### 4. Iterate
NotePlan reloads the plugin when you modify files. No restart needed.

## Essential Patterns

### Always Wrap Async Code in try-catch
```javascript
async function doSomething() {
  try {
    // your code
  } catch (error) {
    console.log('Error: ' + JSON.stringify(error))
  }
}
```

### Query Notes
```javascript
const notes = DataStore.projectNotes
const note = DataStore.projectNoteByTitle('My Note')
const today = DataStore.calendarNoteByDate(new Date())
```

### Read/Write Note Content
```javascript
const paragraphs = note.paragraphs
const tasks = paragraphs.filter(p => p.type === 'open')
paragraphs[2].content = 'Updated text'
note.updateParagraphs(paragraphs)
note.appendParagraph('New item', 'list-item')
```

### User Input
```javascript
const name = await CommandBar.textPrompt('Title', 'Prompt text:')
const choice = await CommandBar.showOptions(DataStore.folders, 'Select a folder')
// choice.value = selected string, choice.index = selected index
```

### Settings
```javascript
const settings = DataStore.settings
DataStore.settings = Object.assign({}, settings, { myKey: 'newValue' })
```

### Create Notes
```javascript
const filename = DataStore.newNote('Note Title', 'FolderName')
// Returns filename string, NOT a note object
const note = DataStore.projectNotes.find(n => n.filename === filename)
```

### Invoke Other Plugin Commands
```javascript
await DataStore.invokePluginCommandByName(
  'Command Name', 'author.PluginId', [arg1, arg2]
)
```

## Plugin Hooks and Triggers

### Hooks
Special function names called automatically (top-level in script.js, no plugin.commands entry needed):

| Function | When Called |
|---|---|
| `init()` | Before any command executes |
| `onUpdateOrInstall()` | After install or update (use for settings migration) |
| `onSettingsUpdated()` | After user saves settings |

### Note Triggers
Notes can trigger plugin commands via frontmatter:
```yaml
---
triggers: onEditorWillSave => yourplugin.id.commandName
---
```
The command must exist in plugin.json (can be `"hidden": true`). Multiple triggers are comma-separated. Supported events: `onEditorWillSave`, `onOpen`.

## Sidebar Pinning (v3.20.1+)

Add `sidebarView` to a command in plugin.json:
```json
{
  "name": "Open My View",
  "jsFunction": "openView",
  "sidebarView": {
    "windowID": "my-custom-id",
    "title": "My Plugin",
    "icon": "calendar",
    "iconColor": "blue-500"
  }
}
```
The `windowID` must match the `id` passed to `HTMLView.showInMainWindow()`. Icon is Font Awesome name, color is Tailwind or hex. NotePlan calls your `jsFunction` when the user clicks the sidebar icon.

## JavaScript Engine Constraints

NotePlan's JS engine is **not Node.js**:
- No `require()`, no `import/export` (unless using the official repo's rollup build)
- No Node built-ins (`fs`, `path`, `Buffer`, etc.)
- Template literals work on macOS 10.15+ / iOS 14+
- `fetch()` available over HTTPS only; use `.then()/.catch()` (not `await`) for error handling
- ES6 features work on macOS 10.15+

For standalone plugins, keep everything in a single script.js file.

## Common Pitfalls

1. **Silent failures**: Async code without try-catch produces no error output.
2. **DataStore.newNote returns filename, not NoteObject**: Look up the note separately.
3. **Editor vs NoteObject**: `Editor` operates on the currently open note only. Use `DataStore` for other notes.
4. **Paragraph positions shift**: After inserting/removing, indices change. Work bottom-to-top or re-fetch.
5. **Hidden Markdown characters**: Prefer paragraph-based operations over raw character positions.
6. **fetch() error handling**: `await fetch()` can't catch errors. Use `.then().catch()`.
7. **No node_modules**: Large folders break sync. Build tooling should bundle into script.js.
8. **Settings defaults**: `DataStore.settings` returns plugin.json defaults. Explicitly setting creates a persistent file.

---

## plugin.json Schema

### Full Example
```json
{
  "noteplan.minAppVersion": "3.20",
  "macOS.minVersion": "10.15.7",
  "iOS.minVersion": "14",
  "plugin.id": "authorid.PluginName",
  "plugin.name": "My Plugin",
  "plugin.description": "What this plugin does.",
  "plugin.author": "Your Name",
  "plugin.url": "https://github.com/...",
  "plugin.version": "1.0.0",
  "plugin.icon": "wrench",
  "plugin.dependencies": [],
  "plugin.script": "script.js",
  "plugin.lastUpdateInfo": "1.0.0: Initial release.",
  "plugin.requiredFiles": [],
  "plugin.commands": [
    {
      "name": "My Command",
      "alias": ["mc", "mycommand"],
      "description": "Short description",
      "jsFunction": "myCommandFunction",
      "arguments": ["argument description"],
      "hidden": false,
      "sidebarView": {
        "windowID": "my-window-id",
        "title": "My Plugin",
        "icon": "calendar",
        "iconColor": "blue-500"
      }
    }
  ],
  "plugin.settings": []
}
```

### Top-Level Fields
| Field | Required | Description |
|---|---|---|
| `noteplan.minAppVersion` | Yes | Minimum NotePlan version. Set based on APIs used. |
| `macOS.minVersion` | No | Defaults to `"10.15.7"`. |
| `iOS.minVersion` | No | `"14"` supports all plugins. |
| `plugin.id` | Yes | Unique `authorid.PluginName`. Must match folder name. |
| `plugin.name` | Yes | Display name in preferences and command bar. |
| `plugin.description` | Yes | Shown in plugin preferences. |
| `plugin.author` | Yes | Author name. |
| `plugin.url` | No | Homepage or repository URL. |
| `plugin.version` | Yes | Semantic version, numbers and periods only. |
| `plugin.icon` | No | Emoji or icon. |
| `plugin.dependencies` | No | Array of JS filenames to load as dependencies. |
| `plugin.script` | Yes | JavaScript filename (`"script.js"`). |
| `plugin.lastUpdateInfo` | No | Changelog shown on auto-update. |
| `plugin.requiredFiles` | No | Files from `requiredFiles/` subfolder. No subdirectories. |
| `plugin.commands` | Yes | Array of command definitions (minimum 1). |
| `plugin.settings` | No | Array of setting definitions. |

### Command Fields
| Field | Required | Description |
|---|---|---|
| `name` | Yes | Command name (without `/`). |
| `alias` | No | Alternative search keywords. |
| `description` | Yes | Shown in Command Bar. |
| `jsFunction` | Yes | Function name in script.js. |
| `arguments` | No | Argument descriptions. |
| `hidden` | No | `true` hides from Command Bar. |
| `sidebarView` | No | Sidebar pinning config (v3.20.1+). |

### sidebarView Fields
| Field | Description |
|---|---|
| `windowID` | Must match `id` in `HTMLView.showInMainWindow()`. Auto-generated if omitted. |
| `title` | Sidebar label. Falls back to `plugin.name`. |
| `icon` | Font Awesome icon name. |
| `iconColor` | Tailwind name or hex. |

### Settings Schema
Each entry in `plugin.settings`:

| Field | Required | Description |
|---|---|---|
| `key` | Yes* | Setting key. *Not needed for `separator`/`heading`. |
| `type` | Yes | Control type (see below). |
| `title` | No | Label above control. |
| `description` | No | Help text. |
| `default` | No | Default value. |
| `required` | No | `true` prevents saving without value. |
| `choices` | No | Array of strings for dropdown. |

### Setting Types
| Type | Control | Value |
|---|---|---|
| `string` | Text input | `string` |
| `[string]` | Comma-separated input | `Array<string>` |
| `number` | Number input | `number` |
| `bool` | Checkbox | `boolean` |
| `date` | Date picker | `string` |
| `json` | Text area | `Object` |
| `separator` | Horizontal line | Visual only |
| `heading` | Section heading | Visual only |
| `hidden` | Invisible | `string` |

### Settings in Code
```javascript
function getSettings() {
  const defaults = { compactMode: false, refreshInterval: 5 }
  return Object.assign({}, defaults, DataStore.settings || {})
}
```

Always merge with explicit defaults in code.

### Settings Migration
```javascript
function onUpdateOrInstall() {
  const settings = DataStore.settings || {}
  if (settings.oldKey !== undefined) {
    settings.newKey = settings.oldKey
    delete settings.oldKey
    DataStore.settings = settings
  }
}
```

### ID Conventions
- Format: `authorid.PluginName` (e.g., `jgclark.Dashboard`, `np.Templating`)
- `np.` prefix is reserved for official plugins
- Command names: title case, unique across ecosystem

---

## API Reference: NotePlan

| Property | Type | Description |
|---|---|---|
| `environment.platform` | `string` | `"macOS"` or `"iOS"` |
| `environment.versionNumber` | `string` | e.g. `"3.20.1"` |
| `environment.buildVersion` | `number` | Build number |
| `environment.colorScheme` | `string` | `"light"` or `"dark"` |
| `environment.languageCode` | `string` | e.g. `"en"` |
| `environment.regionCode` | `string` | e.g. `"US"` |
| `environment.timezone` | `string` | e.g. `"America/New_York"` |

| Method | Returns | Description |
|---|---|---|
| `openURL(url)` | `void` | Open URL in default browser |

## API Reference: Editor

Access to the currently opened note. Shares paragraph methods with NoteObject.

### Properties
| Property | Type | Description |
|---|---|---|
| `content` | `?string` | Full markdown text (getter/setter) |
| `title` | `?string` | First line / title |
| `filename` | `?string` | Filename including path |
| `type` | `?string` | `"Calendar"` or `"Notes"` |
| `paragraphs` | `Array<ParagraphObject>` | All paragraphs |
| `note` | `?NoteObject` | The underlying NoteObject |
| `selectedLineIndex` | `?number` | Current cursor line index |
| `selectedText` | `?string` | Currently selected text |
| `selectedParagraphs` | `Array<ParagraphObject>` | Paragraphs in selection |

### Methods
| Method | Returns | Description |
|---|---|---|
| `insertTextAtCursor(text)` | `void` | Insert at cursor |
| `replaceSelectionWithText(text)` | `void` | Replace selected text |
| `openNoteByFilename(filename)` | `Promise<NoteObject>` | Open note |
| `openNoteByTitle(title)` | `Promise<NoteObject>` | Open by title |
| `openNoteByDate(date)` | `Promise<NoteObject>` | Open calendar note |
| `openNoteByDateString(dateStr)` | `Promise<NoteObject>` | By string (YYYYMMDD) |
| `appendParagraph(text, type)` | `void` | Append paragraph |
| `insertParagraph(text, type, lineIndex)` | `void` | Insert at index |
| `prependParagraph(text, type)` | `void` | Prepend paragraph |
| `updateParagraph(paragraph)` | `void` | Update single |
| `updateParagraphs(paragraphs)` | `void` | Batch update |
| `removeParagraph(paragraph)` | `void` | Remove single |
| `removeParagraphs(paragraphs)` | `void` | Batch remove |
| `highlight(paragraph)` | `void` | Scroll to paragraph |
| `highlightByIndex(lineIndex)` | `void` | Highlight by line |
| `select(start, length)` | `void` | Set selection |
| `renderedSelect(start, length)` | `void` | Selection in rendered positions |

### Paragraph Types (for insert/append)
`'title'`, `'body'`, `'list-item'`, `'task'`, `'done'`, `'scheduled'`, `'cancelled'`, `'quote'`, `'code'`, `'separator'`, `'heading'`

## API Reference: DataStore

### Properties
| Property | Type | Description |
|---|---|---|
| `projectNotes` | `Array<NoteObject>` | All non-calendar notes |
| `calendarNotes` | `Array<NoteObject>` | All calendar notes |
| `folders` | `Array<string>` | All folder paths |
| `settings` | `Object` | Plugin settings (getter/setter) |
| `defaultFileExtension` | `string` | `"md"` or `"txt"` |
| `preference(key)` | `any` | Read a NotePlan preference |

### Methods
| Method | Returns | Description |
|---|---|---|
| `projectNoteByTitle(title)` | `?NoteObject` | Find by exact title |
| `projectNoteByFilename(filename)` | `?NoteObject` | Find by filename |
| `calendarNoteByDate(date)` | `?NoteObject` | Calendar note for date |
| `calendarNoteByDateString(dateStr)` | `?NoteObject` | By string (YYYYMMDD) |
| `newNote(title, folder)` | `?string` | Create note, returns **filename** |
| `newNoteWithContent(content, folder, filename)` | `?string` | Create with content |
| `moveNote(filename, newFolder)` | `?string` | Move to folder |
| `loadJSON(filename)` | `?Object` | Load JSON from plugin folder |
| `saveJSON(object, filename)` | `boolean` | Save JSON to plugin folder |
| `invokePluginCommandByName(name, pluginId, args)` | `Promise<any>` | Call another plugin |
| `listPlugins()` | `Array<PluginObject>` | All installed plugins |

## API Reference: CommandBar

All return Promises, use `await`.

| Method | Returns | Description |
|---|---|---|
| `showOptions(options, title)` | `Promise<{value, index}>` | Fuzzy-searchable list |
| `textPrompt(title, message, defaultText?)` | `Promise<?string>` | Text input |
| `prompt(title, message)` | `Promise<void>` | OK dialog |
| `showInput(placeholder, submitText)` | `Promise<?string>` | Input with submit |

## API Reference: Calendar

| Method | Returns | Description |
|---|---|---|
| `add(calendarItem)` | `Promise<string>` | Create event/reminder |
| `update(calendarItem)` | `Promise<void>` | Update existing |
| `remove(calendarItem)` | `Promise<void>` | Delete |
| `eventsBetween(start, end)` | `Array<CalendarItem>` | Query events |
| `remindersBetween(start, end)` | `Array<CalendarItem>` | Query reminders |
| `parseDateText(text)` | `{start, end}` | Parse natural date text |

## API Reference: Clipboard

| Property | Type | Description |
|---|---|---|
| `string` | `string` | Get/set clipboard text |

## API Reference: HTMLView

| Method | Returns | Description |
|---|---|---|
| `showInMainWindow(html, title, options)` | `Promise<void>` | Full HTML view |
| `showSheet(html, title)` | `Promise<void>` | Modal sheet |
| `showWindow(html, title)` | `Promise<void>` | Floating window |
| `runJavaScript(code, windowId)` | `Promise<any>` | Execute JS in WebView |

### showInMainWindow Options
```javascript
await HTMLView.showInMainWindow(html, 'Title', {
  id: 'my-window-id',
  savedFilename: 'cache.html',
  width: 800, height: 600,
  shouldFocus: true,
  reuseExisting: true
})
```

## API Reference: Types

### NoteObject
| Property | Type | Description |
|---|---|---|
| `title` | `?string` | Note title |
| `filename` | `string` | Full filename with path |
| `type` | `string` | `"Calendar"` or `"Notes"` |
| `content` | `?string` | Full markdown (getter/setter) |
| `paragraphs` | `Array<ParagraphObject>` | All paragraphs |
| `frontmatterTypes` | `Array<string>` | Frontmatter keys |
| `created` | `Date` | Creation date |
| `changedDate` | `Date` | Last modified |
| `hashtags` | `Array<string>` | All #tags |
| `mentions` | `Array<string>` | All @mentions |
| `linkedNotes` | `Array<NoteObject>` | Outgoing links |
| `backlinks` | `Array<NoteObject>` | Incoming links |
| `datedTodos` | `Array<ParagraphObject>` | Todos with dates |

Methods: same paragraph manipulation as Editor (`appendParagraph`, `insertParagraph`, `prependParagraph`, `updateParagraph`, `updateParagraphs`, `removeParagraph`, `removeParagraphs`).

### ParagraphObject
| Property | Type | Description |
|---|---|---|
| `content` | `string` | Text (without markdown prefix) |
| `rawContent` | `string` | Full raw text |
| `type` | `string` | Paragraph type (see below) |
| `lineIndex` | `number` | Line number |
| `heading` | `string` | Parent heading |
| `headingLevel` | `number` | 0-6 |
| `indents` | `number` | Indentation level |
| `isCompleted` | `boolean` | Task completion state |
| `noteType` | `string` | Parent note type |
| `filename` | `string` | Parent filename |

Types: `'title'`, `'body'`, `'open'`, `'done'`, `'scheduled'`, `'cancelled'`, `'checklist'`, `'checklistDone'`, `'checklistScheduled'`, `'checklistCancelled'`, `'list-item'`, `'heading'`, `'quote'`, `'code'`, `'separator'`, `'empty'`

### CalendarItem
| Property | Type | Description |
|---|---|---|
| `title` | `string` | Event/reminder title |
| `date` | `Date` | Start date |
| `endDate` | `?Date` | End date |
| `isAllDay` | `boolean` | All-day flag |
| `calendarName` | `?string` | Calendar name |
| `type` | `string` | `"event"` or `"reminder"` |
| `notes` | `?string` | Description |
| `url` | `?string` | Associated URL |
| `availability` | `number` | 0=busy, 1=free, 2=tentative |
| `id` | `string` | Unique identifier |

```javascript
const event = CalendarItem.create('Meeting', new Date(), new Date(), 'event', false)
event.calendarName = 'Work'
await Calendar.add(event)
```

### RangeObject
`start` (number), `end` (number), `length` (number).

### PluginObject
`id`, `name`, `description`, `author`, `version` (strings), `commands` (Array of `{name, description, arguments}`).

## API Reference: fetch()

HTTPS only. Use `.then()/.catch()`, not `await`, for error handling.

```javascript
fetch('https://api.example.com/data', {
  method: 'GET',
  headers: { 'Authorization': 'Bearer TOKEN', 'Content-Type': 'application/json' },
  timeout: 5000
})
.then(response => { console.log(response) })
.catch(error => { console.log('Fetch error: ' + error) })
```

Parameters: `method` (string), `headers` (Object), `body` (string), `timeout` (number, default 60000).

---

## HTMLView Architecture

Two independent JavaScript contexts:
1. **Plugin Context** (script.js): NotePlan's JS engine. DataStore, Editor, CommandBar access. No DOM.
2. **WebView Context** (HTML): Browser engine. Full DOM. No direct NotePlan API.

Bridges:
- **WebView to Plugin**: `DataStore.invokePluginCommandByName()` from HTML
- **Plugin to WebView**: `HTMLView.runJavaScript()` from script.js

### Generating HTML
Generate a complete HTML document as a string:
```javascript
function generateHTML(data, settings) {
  const colorScheme = NotePlan.environment.colorScheme
  let html = '<!DOCTYPE html><html><head>'
  html += '<meta charset="UTF-8">'
  html += '<style>' + getCSSForTheme(colorScheme) + '</style>'
  html += '</head><body>' + getBodyHTML(data)
  html += '<script>' + getJavaScript(data, settings) + '</script>'
  html += '</body></html>'
  return html
}
```

**Always use JSON.stringify** when injecting data into HTML strings. Never concatenate raw objects.

## HTMLView Communication Patterns

### WebView to Plugin
Create a hidden command in plugin.json, then call from HTML:
```javascript
// In HTML <script>
function sendAction(action, payload, noteFilename) {
  DataStore.invokePluginCommandByName(
    'Handle Action', 'author.PluginId',
    [action, payload, noteFilename]
  )
}
```

Handle in script.js:
```javascript
async function handleAction(action, payload, noteFilename) {
  try {
    if (action === 'save') {
      const data = JSON.parse(payload)
      // modify note...
    }
  } catch (error) {
    console.log('Action error: ' + String(error))
  }
}
```

### Plugin to WebView
```javascript
await HTMLView.runJavaScript(
  'updateUI(' + JSON.stringify(newData) + ');', WINDOW_ID
)
```

The WebView must define the receiving function:
```javascript
function updateUI(data) {
  document.getElementById('content').innerHTML = renderItems(data)
}
```

### Settings from WebView
Route changes through a plugin command rather than writing `DataStore.settings` directly from WebView (can be unreliable):
```javascript
function saveSetting(key, value) {
  DataStore.invokePluginCommandByName(
    'Handle Action', 'author.PluginId',
    ['updateSetting', JSON.stringify({key: key, value: value}), '']
  )
}
```

## HTMLView Theme Integration

### CSS Variable Pattern
```css
:root {
  --bg-main: #ffffff; --bg-card: #f5f5f7; --text-primary: #1d1d1f;
  --text-secondary: #6e6e73; --border-color: #d2d2d7;
  --accent: #007aff; --shadow: rgba(0,0,0,0.08);
}
```

Generate dark variant conditionally based on `NotePlan.environment.colorScheme === 'dark'`:
```javascript
function getCSSVariables(colorScheme) {
  if (colorScheme === 'dark') {
    return ':root { --bg-main: #1c1c1e; --bg-card: #2c2c2e; ' +
      '--text-primary: #f5f5f7; --text-secondary: #98989d; ' +
      '--border-color: #48484a; --accent: #0a84ff; --shadow: rgba(0,0,0,0.3); }'
  }
  return ''
}
```

### Apple Design Language
- Font: `-apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif`
- Subtle shadows, 8-12px border radius, muted colors
- `var(--undefined-variable)` silently produces `transparent`, always verify variable names

## HTMLView Auto-Refresh Pattern

### WebView Side
```javascript
var refreshInterval = null;
function startAutoRefresh(seconds) {
  stopAutoRefresh()
  if (seconds > 0) {
    refreshInterval = setInterval(function() {
      if (!isUserEditing()) {
        DataStore.invokePluginCommandByName(
          'Handle Action', 'author.PluginId',
          ['refresh', '', currentNoteFilename]
        )
      }
    }, seconds * 1000)
  }
}
function stopAutoRefresh() {
  if (refreshInterval) { clearInterval(refreshInterval); refreshInterval = null }
}
function isUserEditing() {
  return !!document.querySelector('.edit-input, .add-input, .popover.open, .picker.open')
}
```

### Key Considerations
1. **Pause during interaction**: Skip refresh while inputs are focused, drag in progress, or dropdowns open.
2. **Refresh all dynamic data**: Not just the active item.
3. **Handle item disappearance**: If active note is deleted, deactivate gracefully.
4. **Configurable interval**: Expose as a setting (3-5 seconds reasonable).

## HTMLView Production Checklist

- [ ] try-catch all async functions (both script.js and WebView)
- [ ] Test light and dark themes
- [ ] Test with no data (empty states show helpful messages)
- [ ] Auto-refresh pauses during interaction and updates all dynamic data
- [ ] Handle item deletion/removal gracefully
- [ ] JSON.stringify all data crossing the bridge
- [ ] console.log key events (prefix with plugin name)
- [ ] Settings survive restart
- [ ] No node_modules or large files
- [ ] Test on both macOS and iOS if targeting both
- [ ] Sidebar pinning works (windowID matches, icon displays)

---

## File Organization

### Standalone (recommended for simplicity)
```
authorid.PluginName/
├── plugin.json
└── script.js
```

### Official GitHub repo tooling
```
authorid.PluginName/
├── plugin.json
├── src/
│   ├── index.js
│   └── helpers.js
├── requiredFiles/
└── __tests__/
```

Rollup bundles `src/` into `script.js`. Node 14 or 16 required. Setup: `npm install && npm run init`. Dev: `npc plugin:dev yourpluginid.Name --watch`.

## Publishing

### Local Distribution
Drop folder in Plugins directory, or zip and share manually.

### Official Repository
Submit PR to [NotePlan/plugins](https://github.com/NotePlan/plugins). Steps:
1. Fork and clone the repo
2. `npm install && npm run init`
3. `npc plugin:create` (scaffolds plugin)
4. `npc plugin:dev yourpluginid.Name --watch` (develop)
5. `npc plugin:info --check yourcommandname` (verify uniqueness)
6. `npc plugin:pr` (submit PR)
7. Admin reviews, merges, and creates a release

### Release Mechanics
NotePlan fetches GitHub Releases, reads `plugin.json` from attached files, compares version against installed plugins, shows Install/Update buttons. Tag format: `pluginid-vX.Y.Z`.

## Requirements for Acceptance

**Must have**: unique `plugin.id` matching folder name, unique command names, valid plugin.json, working script.js, no `node_modules` or large folders, no subdirectories in `requiredFiles`.

**Should have**: meaningful description, populated `plugin.lastUpdateInfo`, `plugin.url`, appropriate `noteplan.minAppVersion`, try-catch on all async, console logging with plugin prefix, settings defaults in code.

**Official repo conventions**: Flow type annotations, ESLint/Prettier config, `/helpers` for shared utilities, source in `src/`, tests in `__tests__/` (Jest).

### Community Resources
- **Discord**: `#plugin-dev` and `#plugin` channels
- **GitHub Issues**: [github.com/NotePlan/plugins/issues](https://github.com/NotePlan/plugins/issues)
- **Plugin Ideas**: [feedback.noteplan.co/plugins-scripting](https://feedback.noteplan.co/plugins-scripting)
