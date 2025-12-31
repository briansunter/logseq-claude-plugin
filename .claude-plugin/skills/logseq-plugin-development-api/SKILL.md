---
name: logseq-plugin-development-api
description: Develop Logseq plugins using @logseq/libs for DB graphs. Use when building Logseq plugins, working with @logseq/libs APIs, querying graph data with Datascript, or creating custom UI for Logseq. Updated for Logseq DB version (2025) with typed properties and unified nodes.
keywords:
- logseq
- plugin
- api
- sdk
- datascript
- db-graph
topics:
- plugin-development
- logseq-api
- typescript
- javascript
---

# Logseq Plugin Development (DB Version)

Develop Logseq plugins using `@logseq/libs` for DB graphs with typed properties, tags (classes), and unified nodes.

**📖 Reference**: [`references/INDEX.md`](references/INDEX.md)

## When to Use

Activate this skill when:

- Developing Logseq plugins
- Working with Logseq DB API (`@logseq/libs`)
- Querying blocks, pages, or tags
- Creating custom UI or commands
- Manipulating Logseq graph data
- Using advanced queries (Datascript/Datalog)

## Quick Start

### Plugin Structure

```
my-plugin/
├── package.json
├── index.html
├── index.ts (or index.js)
└── icon.svg
```

### Minimal Plugin

```typescript
import '@logseq/libs'

async function main() {
console.log('Plugin loaded:', logseq.baseInfo.id)
}

logseq.ready(main).catch(console.error)
```

### package.json

```json
{
"name": "my-plugin",
"main": "index.ts",
"logseq": {
"id": "my-plugin",
"icon": "./icon.svg"
}
}
```

## Core Concepts

### All APIs are Asynchronous

Every `logseq.*` call returns a Promise:

```typescript
// ✅ Correct
const page = await logseq.Editor.getCurrentPage()

// ❌ Wrong - won't work
const page = logseq.Editor.getCurrentPage()
```

### Plugin Lifecycle

```typescript
import '@logseq/libs'

async function main() {
// Plugin is ready
console.log('Plugin loaded:', logseq.settings)

// Register UI
await logseq.App.registerUIItem('toolbar', {
key: 'my-button',
template: `<button>My Button</button>`,
})

// Register command
logseq.App.registerCommand(
'my-plugin:hello',
{
key: 'hello',
},
async () => {
await logseq.UI.showMsg('Hello!')
}
)
}

logseq.ready(main).catch(console.error)
```

### Settings Management

```typescript
// Get settings
const settings = logseq.settings;

// Update settings
logseq.updateSettings({ apiKey: 'xxx' });

// Settings schema (package.json)
{
"logseq": {
"settings": [
{
"key": "apiKey",
"type": "string",
"title": "API Key",
"description": "Enter the API key"
}
]
}
}
```

## Common Patterns

### Reading Pages and Blocks

```typescript
// Get current page
const page = await logseq.Editor.getCurrentPage()
console.log(page.name, page.uuid)

// Get current block
const block = await logseq.Editor.getCurrentBlock()
console.log(block.content, block.uuid)

// Get all pages
const pages = await logseq.Editor.getAllPages()

// Search blocks
const blocks = await logseq.DB.q(`[:find (pull ?b [*])\
:where [?b :block/content ?content]\
[(clojure.string/includes? ?content "todo")]`)
```

### Creating Content

```typescript
// Create page
await logseq.Editor.createPage('My Page', {
properties: { type: 'note' },
})

// Append block to current page
await logseq.Editor.appendBlockInPage(logseq.baseInfo.currentBlock.uuid, 'New block content')

// Insert block
await logseq.Editor.insertBlock('My block', {
sibling: true,
})

// Edit block
await logseq.Editor.updateBlock(blockUuid, 'Updated content')
```

### Working with Properties

```typescript
// Read property
const props = await logseq.Editor.getBlockProperties(blockUuid)
console.log(props['custom-prop'])

// Set property (DB version - typed properties)
await logseq.Editor.upsertBlockProperty(blockUuid, 'status', 'done')

// Get all blocks with property
const blocks = await logseq.DB.q(`
[:find (pull ?b [*])
:where [?b :block/properties ?props]
[(get ?props :status) ?s]
[(= ?s "done")]
]`)
```

### Custom UI

```typescript
// Register toolbar button
await logseq.App.registerUIItem('toolbar', {
key: 'my-button',
template: `
<button data-on-click="handleClick">
My Button
</button>
`,
})

// Register panel
await logseq.App.registerUIItem('panel', {
key: 'my-panel',
template: '<div>My Panel Content</div>',
})

// CSS injection
logseq.provideStyle(`
.my-plugin-button {
background: blue;
}
`)
```

### Queries

```typescript
// Simple query
const result = await logseq.DB.q(`
[:find (pull ?b [*])
:where [?b :block/page ?p]
[?p :block/name "My Page"]]
`)

// Advanced query with properties
const tagged = await logseq.DB.q(`
[:find (pull ?b [*])
:where [?b :block/properties ?props]
[(get ?props :tags) ?tags]
[(clojure.string/includes? ?tags "important")]
`)

// Using Query Builder
const query = logseq.DB.datascriptQuery()
query.find(['?b', '*']).where(['?b', 'block/content'])
const results = await query.exec()
```

## Key APIs

### Editor API

- `getCurrentPage()` - Get current page
- `getCurrentBlock()` - Get current block
- `appendBlockInPage()` - Add block to page
- `insertBlock()` - Insert new block
- `updateBlock()` - Edit block content
- `deleteBlock()` - Remove block
- `getAllPages()` - List all pages

### DB API

- `q()` - Execute Datascript query
- `datascriptQuery()` - Query builder
- `getBlockProperties()` - Read properties
- `upsertBlockProperty()` - Set property
- `addEventListener()` - Listen to changes

### App API

- `registerUIItem()` - Add UI elements
- `registerCommand()` - Register slash command
- `showMsg()` - Show notification
- `showSettings()` - Open settings
- `getCurrentGraph()` - Get graph info

### UI API

- `showMsg()` - Toast notification
- `openModal()` - Modal dialog
- `setPanel()` - Update panel content

## DB Version Changes (2025)

### Typed Properties

```typescript
// Old: free-form properties
// New: typed property values

// String property
await logseq.Editor.upsertBlockProperty(uuid, 'name', 'value')

// Number property
await logseq.Editor.upsertBlockProperty(uuid, 'count', '42')

// Property as data
const props = await logseq.Editor.getBlockProperties(uuid)
console.log(props['name']) // 'value'
```

### Tags as Classes

```typescript
// Tags are now first-class entities
const tagBlocks = await logseq.DB.q(`
[:find (pull ?t [*])
:where [?t :block/type ?t]
[(= ?t :type/tag)]
]`)
```

### Unified Nodes

```typescript
// Pages and blocks use same structure
const entity = await logseq.DB.q(`
[:find (pull ?e [*])
:where [?e :block/uuid ?uuid]]
`)
```

## Debugging

```typescript
// Enable debug mode
logseq.settings.debugMode = true

// Console logging
console.log('Page:', page)
console.log('Block:', block)

// Query result inspection
const result = await logseq.DB.q('[:find ?b :where [?b :block/content]]')
console.log('Query result:', result)
```

## Resources

### Reference

- `REFERENCE.md` - Complete API reference
- `TYPES.md` - TypeScript type definitions
- `EXAMPLES.md` - Code examples

### External Resources

- [Logseq Plugin Docs](https://github.com/logseq/logseq/tree/master/plugins)
- [@logseq/libs API](https://github.com/logseq/logseq/tree/master/packages/libs)
- [Logseq Community](https://discord.gg/logseq)

### Development

- Use TypeScript for type safety
- Test with Logseq developer mode
- Check console for errors
- Use Datascript queries for complex data access

## Learning Path

1. **Start here**: Plugin setup, basic CRUD operations
2. **Then**: Working with pages, blocks, properties
3. **Then**: Queries (Datascript/Datalog)

## Examples

<!-- Add usage examples here -->

4. **Then**: Custom UI, commands, templates
5. **Advanced**: Complex queries, state management, performance