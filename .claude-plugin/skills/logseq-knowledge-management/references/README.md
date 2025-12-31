# Logseq DB Version Reference Documentation

This directory contains reference documentation from the official Logseq docs repository, organized for use with the
Logseq knowledge management skill.

## 🎯 Quick Start for DB Graphs (2025+)

**Start here:** `db-version.md` - Complete DB graph functionality overview

**Migration guide:** `db-version-changes.md` - Changes from file graphs

## 📚 Documentation Guide

### DB Version (Primary Sources)

These files describe the current DB version (2025+) functionality:

| File                     | Description                                                                                                                              |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `db-version.md`          | **START HERE** - Complete DB graph overview with nodes, properties, new tags, tasks, queries, templates, views, library, MCP server, CLI |
| `db-version-changes.md`  | Migration guide - attribute renames, deprecated features, differences from file graphs                                                   |
| `Class.md`               | New Tags (Classes) system with inheritance and properties                                                                                |
| `Property.md`            | Properties documentation (minimal - see db-version.md)                                                                                   |
| `Built-in Properties.md` | Built-in properties reference                                                                                                            |
| `Query.md`               | Simple queries (minimal stub)                                                                                                            |
| `Query Builder.md`       | Visual query builder                                                                                                                     |
| `Query function.md`      | Query function reference                                                                                                                 |
| `Commands.md`            | Slash commands reference                                                                                                                 |
| `Advanced commands.md`   | Advanced commands                                                                                                                        |
| `Assets alias.md`        | Asset management                                                                                                                         |
| `Flashcards.md`          | Spaced repetition flashcards                                                                                                             |
| `Cloze.md`               | Cloze deletion cards                                                                                                                     |
| `Export.md`              | Export options                                                                                                                           |
| `All pages.md`           | Pages view documentation                                                                                                                 |
| `Templates___Docs.md`    | Template examples                                                                                                                        |

### File Graph (Legacy - with Notices)

These files contain **file graph documentation** but include prominent notices about DB version differences:

| File                  | Notice Added      | Key Differences Documented                          |
| --------------------- | ----------------- | --------------------------------------------------- |
| `Tasks.md`            | ✅ Full header    | Status values, priorities, repeaters, time tracking |
| `Templates.md`        | ✅ Full header    | Template creation, dynamic variables, auto-apply    |
| `Advanced Queries.md` | ✅ Warning notice | Attribute renames, deprecated options               |
| `Tags.md`             | ⚠️ Stub only      | See Class.md for DB version                         |
| `templates.md`        | ⚠️ Not checked    | May need review                                     |

## 🔑 Key DB Version Concepts

### Nodes (Unified Blocks/Pages)

- **References:** Both use `[[]]` syntax (blocks no longer use `(())`)
- **Properties:** Both can have typed properties
- **Tags:** Both can have new tags (classes)
- **Timestamps:** Both have `createdAt` and `updatedAt`

### New Tags (Classes)

```markdown
# Create a tag

#Person with properties: lastName, birthday

# Parent tags (inheritance)

#AudioBook extends: #Book, #MediaObject
```

### Typed Properties

| Type     | Example         | Use Case                            |
| -------- | --------------- | ----------------------------------- |
| Text     | "Any text"      | Default, supports refs and children |
| Number   | 2024, 3.5       | Stored as actual numbers            |
| Date     | Journal page ID | Links to journal                    |
| DateTime | Timestamp       | Full datetime with repeaters        |
| Checkbox | true/false      | Boolean values                      |
| URL      | https://...     | URLs only                           |
| Node     | Page/block ID   | References to other nodes           |

### Tasks (DB Version)

```markdown
# Create task

- My task /todo # Uses Status property

# Task properties

Status: Backlog, Todo, Doing, In Review, Done, Canceled Priority: high, medium, low Deadline: <date picker> Scheduled:
<date picker with repeat option>

# Repeated tasks

1. Add Deadline/Scheduled property
2. Check "Repeat task" in date picker
3. Set interval (Day/Week/Month/Year)
4. When marked Done → resets to Todo with new date
```

### Queries (DB Version)

**Simple Queries:**

```clojure
(property status done)
(tags Task)
(and (tags Task) (property priority high))
```

**Advanced Queries (attribute changes):**

```clojure
# Find tasks (DB version)
{:query [:find (pull ?b [*])
         :where
         [?b :block/tags ?t]
         [?t :db/ident :logseq.class/Task]
         [?b :logseq.property/status ?s]]}
```

**Attribute Mapping:**

| Old (File Graph)   | New (DB Graph)                           |
| ------------------ | ---------------------------------------- |
| `:block/content`   | `:block/title`                           |
| `:block/marker`    | `:logseq.property/status`                |
| `:block/priority`  | `:logseq.property/priority`              |
| `:block/deadline`  | `:logseq.property/deadline`              |
| `:block/scheduled` | `:logseq.property/scheduled`             |
| `:block/journal?`  | `[?p :block/tags :logseq.class/Journal]` |

### Templates (DB Version)

```markdown
# Create template

- Weekly Review #Template Apply template to tags:: #Journal
  - ## Wins
    -
  - ## Challenges
    -

# Use template

/Template command → select "Weekly Review"
```

## 🚨 When to Use File Graph Docs

The file graph documentation (with notices) is useful for:

- Understanding legacy syntax when migrating
- Working with existing file graphs
- Learning general concepts that apply to both versions

**Always check `db-version-changes.md`** for the DB version equivalents.

## 📖 Recommended Reading Order

1. **New to DB Graphs:**
   - `db-version.md` (overview)
   - `db-version-changes.md` (if migrating from file graphs)
   - `Class.md` (new tags system)
   - Experiment with queries, tasks, templates

2. **Plugin Developers:**
   - See `../logseq-plugin-development/` for API documentation
   - `Advanced Queries.md` (with DB version notice)
   - `Query function.md`

3. **Migrating from File Graphs:**
   - `db-version-changes.md` (start here!)
   - `Tasks.md` (notice shows status/priority mapping)
   - `Templates.md` (notice shows template syntax changes)
   - `Advanced Queries.md` (notice shows attribute renames)

## 🔍 Finding Specific Information

- **Tasks:** `Tasks.md` (has DB version notice) or `db-version.md#tasks`
- **Queries:** `Query.md` (stub) → see `db-version.md#queries` and `Advanced Queries.md`
- **Templates:** `Templates.md` (has DB version notice) or `db-version.md#templates`
- **Properties:** `Property.md` (minimal) → see `db-version.md#properties`
- **Tags/Classes:** `Class.md` or `db-version.md#new-tags`

## ⚠️ Important Notes

1. **DB graphs only support Markdown** - Org mode is file graph only
2. **Properties are typed** - Define types before use with `upsertProperty`
3. **All `logseq.*` API calls are async** - Always use `await`
4. **Date properties store journal page IDs** - Not date strings!
5. **Dynamic variables (`<% today %>`) are WIP** in DB graphs

## 📝 Contributing

When adding new reference documentation:

1. Copy from official Logseq docs repository
2. If it's file graph content, add a DB version notice
3. Update this README with the new file
4. Consider whether content needs adaptation for DB version

## 🌐 Official Documentation

- Official docs: https://github.com/logseq/docs
- DB version test: https://test.logseq.com/
- Community: https://discuss.logseq.com/

---

**Last updated:** 2025-12-27 **Logseq DB Version:** December 19, 2025
