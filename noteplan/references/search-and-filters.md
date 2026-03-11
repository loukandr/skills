# NotePlan Search and Filters Reference

## Table of Contents

- [Search Syntax](#search-syntax)
- [Search Operators](#search-operators)
- [Predefined Filters](#predefined-filters)
- [Custom Filters](#custom-filters)

---

## Search Syntax

### Basic Patterns

| Pattern | Behavior |
|---|---|
| `word` | Matches lines containing the word |
| `"word"` | Exact/whole word match (quoted) |
| `term1 term2` | Implicit AND (both must match) |
| `term1 OR term2` | Boolean OR (either matches) |
| `-term` | Exclude lines containing term |
| `-#tag` | Exclude lines with tag |
| `(term1 OR term2)` | Grouping |
| `-(work meetup)` | Grouped negation (exclude lines with both) |

### Complex Examples
```
#pending -#waiting
meeting OR meetup
(projectA (design OR testing)) OR (projectB spec)
("design review" OR "code review") #todo
meeting -(work meetup)
```

## Search Operators

### Source Operators
Filter by note source. Multiple sources with comma: `source:notes,events`

| Operator | Description |
|---|---|
| `source:calendar` | Calendar notes only |
| `source:notes` | Project notes only |
| `source:events` | Calendar events |
| `source:reminders` | Apple Reminders |
| `source:list-reminders` | Reminders by list |
| `source:dated-notes` | Notes with dates |

### Task Status Operators

| Operator | Description |
|---|---|
| `is:open` | Open tasks |
| `is:done` | Completed tasks |
| `is:scheduled` | Scheduled tasks |
| `is:canceled` | Cancelled tasks (also `is:cancelled`) |
| `is:not-task` | Non-task lines |

### Checklist Status Operators

| Operator | Description |
|---|---|
| `is:checklist` | Open checklists |
| `is:checklist-done` | Completed checklists |
| `is:checklist-scheduled` | Scheduled checklists |
| `is:checklist-cancelled` | Cancelled checklists |

### Date Operators (Relative)

| Operator | Description |
|---|---|
| `date:today` | Today |
| `date:yesterday` | Yesterday |
| `date:tomorrow` | Tomorrow |
| `date:past` | All past dates |
| `date:future` | All future dates |
| `date:past-and-today` | Past and today |
| `date:this-week` | Current week |
| `date:last-week` | Previous week |
| `date:next-week` | Next week |
| `date:this-month` | Current month |
| `date:last-month` | Previous month |
| `date:next-month` | Next month |
| `date:this-year` | Current year |
| `date:last-year` | Previous year |
| `date:next-year` | Next year |
| `date:30days` | Next 30 days |
| `date:all` | All dates |

### Date Operators (Specific ISO)

| Format | Example |
|---|---|
| Daily | `date:2026-02-20` |
| Weekly | `date:2026-W08` |
| Monthly | `date:2026-02` |
| Quarterly | `date:2026-Q1` |
| Yearly | `date:2026` |

### Date Ranges
```
date:2026-02-01-2026-02-28
date:2026-W01-2026-W52
date:2026-01-2026-06
date:2026-Q1-2026-Q4
date:2025-2026
```

### Location Operators

| Operator | Example | Description |
|---|---|---|
| `path:` | `path:Projects/Work` | Filter by folder path |
| `path:` | `path:"10 - Projects/Marketing"` | Quoted for spaces |
| `heading:` | `heading:TODO` | Filter by heading name |
| `heading:` | `heading:"Project Goals"` | Quoted for spaces |

### Sort Operators

| Operator | Description |
|---|---|
| `sort:asc` | Ascending order |
| `sort:desc` | Descending order |

### View Options

| Operator | Description |
|---|---|
| `show:timeblocked` | Show time-blocked items |
| `show:past-events` | Show past events |
| `show:archive` | Include archived notes |
| `show:teamspaces` | Include teamspaces/spaces |
| `hide:past-events` | Hide past events |
| `hide:archive` | Exclude archived notes |
| `hide:teamspaces` | Exclude teamspaces |

## Predefined Filters

| Filter | Description |
|---|---|
| All Tasks | Open tasks from all notes |
| Note Tasks | Open/scheduled tasks from project notes only |
| Overdue | Open/scheduled tasks from past dates |
| Upcoming | Tasks due today or in the future |

Access via sidebar or URL scheme: `noteplan://x-callback-url/search?filter=Upcoming`

## Custom Filters

Create custom filters with:
- **Task type**: Open, done, cancelled, scheduled, non-task
- **Search criteria**: Text, hashtags, mentions
- **Origin**: Specific folders, reminders by list
- **Date scope**: This week, this month, etc.

Filters appear in the sidebar and can be accessed via URL scheme.
