# NotePlan URL Scheme Reference

## Table of Contents

- [General Format](#general-format)
- [Actions](#actions)
  - [openNote](#opennote)
  - [openView](#openview)
  - [addText](#addtext)
  - [addNote](#addnote)
  - [deleteNote](#deletenote)
  - [selectTag](#selecttag)
  - [search](#search)
  - [runPlugin](#runplugin)
  - [installPlugin](#installplugin)
  - [toggleSidebar](#togglesidebar)
  - [noteInfo](#noteinfo)
- [x-callback-url Support](#x-callback-url-support)
- [Custom Links](#custom-links)

---

## General Format

```
noteplan://x-callback-url/[action]?[parameters]&[x-callback parameters]
```

All parameter values must be URL-encoded. Test with:
```bash
open "noteplan://x-callback-url/addText?noteDate=20260220&text=Hello%20World"
```

## Actions

### openNote

Open a note by date, title, or filename.

| Parameter | Description |
|---|---|
| `noteDate` | `YYYYMMDD`, `today`, `yesterday`, `tomorrow`, or ISO week `YYYY-Www` |
| `timeframe` | `week`, `month`, `quarter`, `year` (used with noteDate for periodic notes) |
| `noteTitle` | Note title. Append `#heading` to link to a subheading |
| `filename` | Relative path with URL encoding |
| `heading` | Subheading to scroll to and highlight (v3.7.2+) |
| `subWindow` | `yes`/`no` (Mac only) |
| `splitView` | `yes`/`no` (Mac only, v3.4+) |
| `reuseSplitView` | `yes`/`no` (Mac only, v3.20.1+) |
| `useExistingSubWindow` | `yes`/`no` (Mac only, v3.2+) |
| `highlightStart` | Character index for selection start (v3.9+) |
| `highlightLength` | Selection length (v3.9+) |

Examples:
```
noteplan://x-callback-url/openNote?noteDate=today
noteplan://x-callback-url/openNote?noteTitle=My%20Project
noteplan://x-callback-url/openNote?noteDate=20260220&timeframe=week
noteplan://x-callback-url/openNote?noteTitle=My%20Note&heading=Section%202
```

### openView

Open a named folder view (v3.18.1+).

| Parameter | Description |
|---|---|
| `name` | View name (URL-encoded) |
| `folder` | Relative folder path (URL-encoded, optional) |

Example:
```
noteplan://x-callback-url/openView?name=Project%20Tasks&folder=10%20-%20Projects
```

### addText

Add text to an existing note.

| Parameter | Description |
|---|---|
| `noteDate` | Calendar note identifier |
| `noteTitle` | Note title identifier |
| `fileName` | Filename identifier |
| `text` | Content to add (URL-encoded) |
| `mode` | `prepend` or `append` |
| `openNote` | `yes`/`no` |
| `subWindow`, `splitView`, `useExistingSubWindow` | Mac only |

Example:
```
noteplan://x-callback-url/addText?noteDate=today&text=*%20New%20task&mode=append&openNote=yes
```

### addNote

Create a new note.

| Parameter | Description |
|---|---|
| `noteTitle` | Note heading (optional) |
| `text` | Content (optional) |
| `openNote` | `yes`/`no` |
| `folder` | Target folder path |
| `subWindow`, `splitView`, `useExistingSubWindow` | Mac only |
| `highlightStart`, `highlightLength` | Selection (v3.9+) |

Example:
```
noteplan://x-callback-url/addNote?noteTitle=Meeting%20Notes&folder=Projects&openNote=yes
```

### deleteNote

Delete a note.

| Parameter | Description |
|---|---|
| `noteTitle` | Note title |
| `noteDate` | Calendar note date |
| `fileName` | Filename |

Example:
```
noteplan://x-callback-url/deleteNote?noteTitle=Old%20Note
```

### selectTag

Filter by tag or mention.

| Parameter | Description |
|---|---|
| `name` | Tag name (prepend `#` or `@`, URL-encoded). Leave empty to show all. |

Example:
```
noteplan://x-callback-url/selectTag?name=%23noteplan
```

### search

Search notes or open a saved filter.

| Parameter | Description |
|---|---|
| `text` | Search string |
| `filter` | Existing filter name (alternative to text) |

Examples:
```
noteplan://x-callback-url/search?text=noteplan
noteplan://x-callback-url/search?filter=Upcoming
```

### runPlugin

Execute a plugin command.

| Parameter | Description |
|---|---|
| `pluginName` or `pluginID` | Plugin identifier |
| `command` | Command name (without leading `/`) |
| `arg0`, `arg1`, `arg2`... | Optional arguments (v3.5+) |

Example:
```
noteplan://x-callback-url/runPlugin?pluginID=dwertheimer.Favorites&command=nc
```

### installPlugin

Install a plugin (v3.15+).

| Parameter | Description |
|---|---|
| `pluginID` | Plugin identifier |

Example:
```
noteplan://x-callback-url/installPlugin?pluginID=jgclark.Dashboard
```

### toggleSidebar

Control sidebar visibility (v3.19.2+).

| Parameter | Description |
|---|---|
| `forceCollapse` | `yes`/`no` |
| `forceOpen` | `yes`/`no` |
| `animated` | `yes`/`no` (Mac only, defaults yes) |

### noteInfo

Return current note info (v3.5+). Requires `x-success` parameter. Returns `path` and `name` via callback.

## x-callback-url Support

Every action supports `x-success` (v3.5+) to return to the calling app after processing:
```
noteplan://x-callback-url/addText?noteDate=today&text=Hello&x-success=sourceapp://x-callback-url
```

## Custom Links

Define custom link patterns via theme JSON using regex that detect text and transform it into clickable links. Each custom link has:
- `regex`: Pattern to match
- `matchPosition`: Capture group to make clickable
- `urlPosition`: Capture group for the URL
- `type`: `noteLink` or `link`
- `prefix`: URL prefix to prepend
