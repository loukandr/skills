# NotePlan Templates Reference

## Table of Contents

- [Overview](#overview)
- [Template Types](#template-types)
- [Template Frontmatter](#template-frontmatter)
- [Tag Syntax](#tag-syntax)
- [Date Tags](#date-tags)
- [Prompt Tags](#prompt-tags)
- [Web Service Tags](#web-service-tags)
- [Meeting Note Tags](#meeting-note-tags)
- [JavaScript in Templates](#javascript-in-templates)
- [Auto-Insert Templates](#auto-insert-templates)

---

## Overview

Templates reside in a dedicated "Templates" folder under Smart Folders in the sidebar. Create templates by right-clicking (Mac) or long-pressing (iOS) the Templates folder. A template consists of optional frontmatter properties and a body with placeholder tags.

## Template Types

Set via the `type` frontmatter property:

| Type | When shown |
|---|---|
| `empty-note` | When inserting into blank notes |
| `meeting-note` | For meeting note creation |
| `calendar-note` | Daily calendar notes only (v3.5.2+) |
| `project-note` | Project notes only (v3.5.2+) |

## Template Frontmatter

| Field | Description |
|---|---|
| `title` | Template display name |
| `type` | Template type (see above) |
| `folder` | Target folder. Use `<select>` for prompt, `<current>` for current folder |
| `append` | Append to existing note (by title, `<select>`, or `<current>`) |
| `prepend` | Prepend to existing note |
| `newNoteTitle` | For Shortcuts integration |
| `repeat` | Auto-insert scheduling (see Auto-Insert section) |

Use double-dash `--` delimiters (instead of `---`) for raw blocks within templates to avoid conflicts.

## Tag Syntax

| Tag | Purpose |
|---|---|
| `<%-` ... `%>` | Output value into template (most common) |
| `<%` ... `%>` | Execute JavaScript, no output |
| `<%_` ... `%>` | Whitespace-slurping (strips whitespace before) |
| `<%#` ... `%>` | Comment (no execution, no output) |
| `<%%` | Outputs literal `<%` |
| `-%>` | Strips whitespace after tag |

## Date Tags

### Current Date
```
<%- date.now("Do MMMM YYYY") %>
```

### With Offset
```
<%- date.now("Do MMMM YYYY", -2) %>        → 2 days ago
<%- date.now("Do MMMM YYYY", 5) %>         → 5 days from now
<%- date.now("Do MMMM YYYY", "2w") %>      → 2 weeks from now
<%- date.now("Do MMMM YYYY", "2M") %>      → 2 months from now
<%- date.now("Do MMMM YYYY", "2y") %>      → 2 years from now
```

### Format Specific Date
```
<%- date.format("Do MMMM YYYY", "2026-12-01") %>
<%- date.format("Do MMMM YYYY", Editor.title) %>
```

### Advanced Date Functions

| Function | Description |
|---|---|
| `date.weekday(offset)` | Next weekday with offset (skips weekends) |
| `date.weekNumber()` | Current week number |
| `date.dayNumber()` | Day of week (0-6) |
| `date.isWeekday()` | Boolean: is today a weekday |
| `date.isWeekend()` | Boolean: is today a weekend |
| `date.startOfWeek()` | Start of current week |
| `date.endOfWeek()` | End of current week |
| `date.startOfMonth()` | Start of current month |
| `date.endOfMonth()` | End of current month |
| `date.daysInMonth()` | Number of days in current month |
| `date.daysBetween()` | Days between two dates |
| `date.add(amount, unit)` | Add to date |
| `date.subtract(amount, unit)` | Subtract from date |
| `date.businessAdd()` | Add business days |
| `date.businessSubtract()` | Subtract business days |
| `date.nextBusinessDay()` | Next business day |
| `date.previousBusinessDay()` | Previous business day |

### Format Tokens (MomentJS-based)

| Token | Output |
|---|---|
| `YYYY` | 4-digit year |
| `YY` | 2-digit year |
| `MMMM` | Full month name |
| `MMM` | Short month name |
| `MM` | 2-digit month |
| `M` | Month number |
| `Do` | Day with ordinal (1st, 2nd) |
| `DD` | 2-digit day |
| `D` | Day number |
| `dddd` | Full weekday name |
| `ddd` | Short weekday name |
| `HH:mm:ss` | 24-hour time |
| `hh:mm a` | 12-hour time |
| `W`, `WW` | ISO week number |
| `X` | Unix timestamp (seconds) |
| `x` | Unix timestamp (ms) |

## Prompt Tags

### Text Input
```
<%- prompt('What is your first name?') %>
```

### Choice List
```
<%- prompt('variable', 'Question?', ['option1', 'option2']) %>
```

### Date Input
```
<%- promptDate('variable', 'Enter start date:') %>
```
Returns YYYY-MM-DD format.

### Date Interval
```
<%- promptDateInterval('variable', 'Enter interval:') %>
```
Returns format like `nnn[bdwmqy]`.

### Frontmatter Key Values
```
<%- promptKey('key', 'message', 'noteType', caseSensitive, 'folder', fullPathMatch, ['options']) %>
```
Select from existing frontmatter values across notes.

### Tag Selector
```
<%- promptTag('message', 'includePattern', 'excludePattern', allowCreate) %>
```

### Mention Selector
```
<%- promptMention('message', 'includePattern', 'excludePattern', allowCreate) %>
```

Variable names must start with an alpha character and contain only alphanumeric characters.

## Web Service Tags

```
<%- web.weather() %>                                    → Local weather
<%- web.advice() %>                                     → Random advice
<%- web.quote() %>                                      → Random quote
<%- web.services('affirmation', 'affirmation') %>       → Random affirmation
```

Custom web services configurable in Preferences > Plugins > Templating settings. Supports plain text or JSON APIs over HTTPS.

## Meeting Note Tags

| Tag | Description |
|---|---|
| `<%- eventTitle %>` | Calendar event name |
| `<%- eventAttendees %>` | Comma-separated attendees with email links |
| `<%- eventAttendeeNames %>` | Attendee names as plain text |
| `<%- calendarItemLink %>` | Link to the event (required for meeting notes) |
| `<%- eventDate('format') %>` | Event date with format |
| `<%- eventEndDate('format') %>` | Event end date |
| `<%- eventLink %>` | URL attached to event (e.g., Zoom link) |
| `<%- eventNotes %>` | Event's notes field |
| `<%- eventLocation %>` | Event location |
| `<%- eventCalendar %>` | Calendar name |

## JavaScript in Templates

### Inline Tags
For one-liners with full NotePlan Plugin API access:
```
<%- DataStore.projectNotes.length %>
```

### templatejs Code Blocks
For multi-line JavaScript:
````
```templatejs
const notes = DataStore.projectNotes
const count = notes.filter(n => n.hashtags.includes('#project')).length
this.count = count
```
Project count: <%- count %>
````

Variables defined in `templatejs` blocks are available throughout the template via `this.varName` and `<%- varName %>`.

## Auto-Insert Templates

Templates automatically inserted into new calendar notes. Configure via the `repeat:` frontmatter field:

```yaml
repeat: note = day, freq = week, on = [mon, tue, wed, thu, fri]
```

### Parameters

| Parameter | Values | Description |
|---|---|---|
| `note` | `day`, `week`, `month`, `quarter`, `year` | Calendar note type |
| `freq` | `day`, `week`, `month`, `quarter`, `year` | Repetition frequency |
| `on` | Weekday names, numbers 1-31, MM-DD | Specific days/dates |
| `order` | Number | Insertion sequence for multiple templates |
| `async` | `true`/`false` | For templates with web calls or event listings |

### Examples
```yaml
# Weekdays only
repeat: note = day, freq = week, on = [mon, tue, wed, thu, fri]

# First of every month
repeat: note = day, freq = month, on = [1]

# Every December 24th
repeat: note = day, freq = year, on = [12-24]

# Weekly notes every week
repeat: note = week, freq = week
```

Auto-insert only applies to notes that do not yet exist. Modified templates apply only to future new notes.
