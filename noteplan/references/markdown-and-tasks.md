# NotePlan Markdown and Tasks Reference

## Table of Contents

- [Paragraph Types](#paragraph-types)
- [Task States in Detail](#task-states-in-detail)
- [Checklists vs Tasks](#checklists-vs-tasks)
- [Extended Markdown Syntax](#extended-markdown-syntax)
- [Synced Lines](#synced-lines)
- [Attributed Tags](#attributed-tags)
- [Folding](#folding)
- [Custom Markdown via Themes](#custom-markdown-via-themes)

---

## Paragraph Types

NotePlan recognizes these paragraph types internally:

| Type | Markdown | Description |
|---|---|---|
| `title` | `# Heading` | Title/heading (levels 1-4) |
| `body` | Plain text | Regular paragraph |
| `open` | `* task` | Open task |
| `done` | `* [x] task` | Completed task |
| `scheduled` | `* [>] task` | Scheduled/rescheduled task |
| `cancelled` | `* [-] task` | Cancelled task |
| `checklist` | `+ item` | Open checklist item |
| `checklistDone` | `+ [x] item` | Completed checklist |
| `checklistScheduled` | `+ [>] item` | Scheduled checklist |
| `checklistCancelled` | `+ [-] item` | Cancelled checklist |
| `list-item` | `- item` | Bullet point (default dash behavior) |
| `heading` | `## Heading` | Section heading |
| `quote` | `> text` | Blockquote |
| `code` | Triple backticks | Code block |
| `separator` | `---` | Horizontal divider |
| `empty` | (blank line) | Empty line |

## Task States in Detail

### Open Tasks
```
* Buy groceries
* [ ] Buy groceries
```
Both forms are equivalent. The bracket form is explicit.

### Done Tasks
```
* [x] Buy groceries @done(2026-02-20)
```
Completion date appended automatically when "Append Completion Date" is enabled in Preferences > Todo. Format: `@done(YYYY-MM-DD)` or `@done(YYYY-MM-DD HH:mm)`.

### Cancelled Tasks
```
* [-] Buy groceries
```
Use CMD+R to cancel. Cancelled tasks retain their text but are visually struck through.

### Scheduled Tasks
```
* [>] Buy groceries >2026-02-25
```
Created automatically when rescheduling a task. The `[>]` state indicates the task was moved. A back-link `<YYYY-MM-DD` on the new copy references the original date.

## Checklists vs Tasks

Checklists (created with `+`) differ from tasks (`*`) in three key ways:

1. Do not increment the overdue task counter on iOS
2. Do not appear in the overdue list/filters
3. Do not contribute to calendar heat maps

Checklists cannot independently have tags, start dates, or deadlines (those properties belong to the parent task). The Repeat Extensions plugin does not work with checklists because NotePlan never appends completion dates to them.

Common checklist use cases: packing lists, grocery lists, subtask steps, time blocks as sub-items.

## Extended Markdown Syntax

### Standard Markdown
| Syntax | Effect |
|---|---|
| `# Heading` through `#### Heading` | Headings (4 levels) |
| `**bold**` | Bold |
| `*italic*` | Italic |
| `- bullet` | Bullet point |
| `> quote` | Blockquote |
| `` `code` `` | Inline code |
| Triple backticks | Code block (with language specifier) |
| `[text](url)` | Link |
| `![alt](url)` | Image |
| `---` | Horizontal divider |

### NotePlan Extensions
| Syntax | Effect |
|---|---|
| `~~text~~` | Strikethrough |
| `==text==` | Highlighting |
| `::text::` | Highlighting (alternative) |
| `~text~` | Underline |
| `%%comment%%` | Hidden comment (invisible unless cursor is on the line) |

### Tables
Supported since v3.8.1. Full markdown formatting within cells since v3.17.3:
```
| Column 1 | Column 2 |
|----------|----------|
| data     | data     |
```

### Priority Markers
Place at line beginning, before task content:
| Syntax | Priority |
|---|---|
| `!` | Important (low) |
| `!!` | Urgent (medium) |
| `!!!` | Very urgent (high) |
| `>>` | Current focus |

Sort order: `>>` > `!!!` > `!!` > `!`. Rendered with progressively intensified background colors.

## Synced Lines

A synced line is a single line mirrored across notes. Each synced line gets a unique ID appended (e.g., `^129abz`). Changes to any copy propagate to all connected versions.

### Creating Synced Lines
- **Mac/iPad**: CMD+drag a line to another note
- **Context menu**: Right-click > "Copy Synced Line"

### Behavior
- Clicking the sync icon on a synced line shows all notes containing copies
- When rescheduling, only open sub-tasks move forward
- Sync is removed from new copies during rescheduling to prevent cascading completions

### Linking to Synced Lines
```
[[Note Title^blockID]]
```

## Attributed Tags

Available since v3.17. Tags with structured metadata properties:

```
#todo(state: backlog, assigned: Eduard)
#todo(icon: lightbulb, icon-style: solid, icon-color: green-600)
```

Properties enable grouping in Kanban/folder views. The `state` property determines which column a card appears in.

## Folding

Tasks and bullets with indented content beneath can be folded/collapsed:
- **Shortcut**: CMD+/
- **Visual**: Small triangle indicator on foldable lines
- Folding badges show count of hidden items (v3.15.2+)

## Custom Markdown via Themes

Extend markdown by adding custom regex patterns in theme JSON files. Each pattern supports:

| Attribute | Description |
|---|---|
| `regex` | Pattern to match |
| `matchPosition` | Which capture group gets styled |
| `isHiddenWithoutCursor` | Hide matched text when cursor is elsewhere |
| `isRevealOnCursorRange` | Reveal hidden text when cursor enters |
| `isMarkdownCharacter` | Mark delimiters as markdown syntax |
| `backgroundColor` | Background color |
| `color` | Text color |
| `strikethroughStyle` | Strikethrough styling |
| `underlineStyle` | Underline styling |

Limitation: Custom patterns do not work across multiple paragraphs (NotePlan refreshes per paragraph for performance).

### Theme Color Format
All colors use hexadecimal (e.g., `#ffffff`, `#d87001`).

### Theme Structure
Two major sections:
- `editor`: UI colors (backgroundColor, tintColor, textColor, toolbarBackgroundColor, etc.)
- `styles`: Text formatting for elements (body, title1-3, todo, checked, code-fence, etc.)

Customize via Preferences > Themes > hover theme > "Copy & Customize". Changes auto-sync.
