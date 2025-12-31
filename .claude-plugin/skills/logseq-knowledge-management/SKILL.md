---
name: logseq-knowledge-management
description: Use for Logseq knowledge management workflows, graph queries, block manipulation, and PKM (Personal Knowledge
  Management) best practices with Logseq's outliner-based approach.
keywords: [logseq, pkm, knowledge-management, outliner, graph-db, notes, zettelkasten]
topics: [logseq-queries, blocks, pages, properties, tags, pdf-annotation, workflow]
---

# Logseq Knowledge Management

Master Logseq for personal knowledge management with outliner-based workflows, graph databases, and block-based
organization.

## When to Use

Activate this skill when:

- Setting up or using Logseq for PKM
- Writing advanced queries (Datascript/Datalog)
- Organizing knowledge with blocks and pages
- Configuring Logseq settings and plugins
- Implementing Zettelkasten or PARA systems
- Managing PDFs and annotations
- Creating custom workflows and templates

## Quick Start

### Basic Block Structure

```markdown
- Parent block
  - Child block 1
  - Child block 2
    - Nested block
```

### Page Properties

```markdown
title:: My Page tags:: [[tag1]] [[tag2]] type:: documentation status:: active
```

### Basic Query

```clojure
{:query
  [:find (pull ?b [*])
   :where
    [?b :block/page ?p]
    [?p :block/name "My Page"]]}
```

## Core Concepts

### Blocks vs Pages

- **Pages**: Containers for topics (like folders)
- **Blocks**: Atomic units of knowledge (like notes)
- **Hierarchy**: Blocks can nest infinitely
- **References**: Link blocks with `[[page]]` or `((uuid))`

### Properties as Metadata

```markdown
- Task Block status:: active priority:: high due:: 2025-01-15 assigned:: [[John Doe]]
```

### Graph Database

Logseq uses Datascript (Datalog):

- **Entities**: Pages, blocks, users
- **Attributes**: Properties, content, metadata
- **Queries**: Datalog for complex searches

## Common Patterns

### Daily Notes

```markdown
- [[January 15th, 2025]]
  - #meeting Standup
    - Discussed Q1 roadmap
    - Action items:: [[Task A]], [[Task B]]
  - #writing Blog post draft
    - Outline complete
```

### Zettelkasten

```markdown
- Permanent Note type:: evergreen-note tags:: [[concept]]
  - Key insight
    - Connection to [[Other Note]]
```

### Task Management

```markdown
- [[Project X]] status:: active
  - TODO Task 1 status:: todo
  - DONE Task 2 status:: done completed:: [[2025-01-15]]
```

### Queries

```clojure
{:query
  [:find (pull ?b [*])
   :where
    [?b :block/properties ?props]
    [(get ?props :status) ?s]
    [(= ?s :todo)]]
  :title "Active Tasks"}
```

## Key Features

### Advanced Queries

- **Property filters**: Search by property values
- **Tag relationships**: Find connected blocks
- **Date queries**: Time-based searches
- **Boolean logic**: AND, OR, NOT operations

See: `references/Advanced Queries.md`

### Custom Commands

```markdown
- Shortcut: `j` opens journal
- Shortcut: `t` creates todo
- Shortcut: `l` shows linked references
```

See: `references/Commands.md`

### Templates

```markdown
- Book Notes template:: book-note title:: {{title}} author:: {{author}} rating:: {{rating}}
```

See: `references/Templates.md`

## Queries

### Simple Query

```clojure
{:query
  [:find ?b
   :where
    [?b :block/content ?content]
    [(clojure.string/includes? ?content "important")]]
}
```

### Property Query

```clojure
{:query
  [:find (pull ?b [:block/content :block/properties])
   :where
    [?b :block/properties ?props]
    [(get ?props :status) ?s]
    [(= ?s "active")]]
}
```

### Tag Query

```clojure
{:query
  [:find (pull ?p [*])
   :where
    [?p :block/type ?type]
    [(= ?type :type/tag)]
    [?p :block/name ?name]
    [(clojure.string/starts-with? ?name "dev")]]
}
```

## Resources

### References

- `Advanced Queries.md` - Complex query patterns
- `Advanced commands.md` - Custom command reference
- `All pages.md` - Query all pages
- `Assets alias.md` - File attachments
- `Built-in Properties.md` - Standard properties
- `Class.md` - Tag classes
- `Cloze.md` - Flashcard creation
- `Commands.md` - Keyboard shortcuts
- `Config edn file.md` - Configuration
- `custom.css.md` - Styling
- `custom.js.md` - Custom scripts
- `Export.md` - Export options
- `Flashcards.md` - Spaced repetition
- `Properties.md` - Using properties
- `Query function.md` - Query helpers
- `Query Builder.md` - GUI query builder
- `Query.md` - Query basics
- `Tags.md` - Tag system
- `Tasks.md` - Task management
- `Templates.md` - Template creation
- `Templates___Docs.md` - Template docs
- `db-version.md` - Database changes
- `db-version-changes.md` - Version migration

### Examples

- `EXAMPLES.md` - Code examples and workflows

### External Resources

- [Logseq Documentation](references/contents.md)
- [Logseq GitHub](https://github.com/logseq/logseq)
- [Logseq Community](https://discord.gg/logseq)

## Learning Path

1. **Start here**: Blocks, pages, basic linking
2. **Then**: Properties, tags, daily notes
3. **Then**: Basic queries, templates
4. **Then**: Advanced queries, custom commands
5. **Advanced**: Plugins, API, automation
