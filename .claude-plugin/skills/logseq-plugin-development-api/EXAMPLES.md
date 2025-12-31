# Logseq Plugin Development Examples (DB Version)

This file contains practical code examples for developing Logseq plugins using `@logseq/libs` for DB graphs. See
SKILL.md for API documentation and REFERENCE.md for complete reference.

## Table of Contents

- [Setup and Structure](#setup-and-structure)
- [Page and Block Operations](#page-and-block-operations)
- [Properties and Tags](#properties-and-tags)
- [Queries](#queries)
- [UI Integration](#ui-integration)
- [Settings and Storage](#settings-and-storage)
- [Complete Plugin Examples](#complete-plugin-examples)

## Setup and Structure

### Minimal Plugin Structure

```typescript
// package.json
{
  "name": "my-logseq-plugin",
  "version": "0.1.0",
  "logseq": {
    "id": "my-logseq-plugin"
  }
}
```

```typescript
// index.ts
import '@logseq/libs'

async function main() {
  console.log('Plugin loaded!')
}

logseq.ready(main).catch(console.error)
```

### Vite Configuration

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import logseqDevPlugin from 'vite-plugin-logseq'

export default defineConfig({
  plugins: [logseqDevPlugin()],
  build: {
    target: 'esnext',
    minify: 'esbuild',
    lib: {
      entry: './index.ts',
      name: 'myPlugin',
      formats: ['iife'],
      fileName: 'index',
    },
  },
})
```

## Page and Block Operations

### Creating Pages with Tags and Properties

```typescript
// Create a book page with properties
const book = await logseq.Editor.createPage('Great Gatsby', {
  tags: ['Book'],
  author: 'F. Scott Fitzgerald',
  year: 1925,
  rating: 9,
})

// Create a journal entry
const journal = await logseq.Editor.createPage(
  '2025-01-08',
  {
    tags: ['Journal'],
  },
  {
    redirect: false, // Don't navigate to page
  }
)
```

### Working with Blocks

```typescript
// Get current block
const block = await logseq.Editor.getCurrentBlock()
if (!block) {
  logseq.UI.showMsg('No block selected', 'error')
  return
}

// Insert a child block
await logseq.Editor.insertBlock(
  block.uuid,
  'New child content',
  { sibling: false } // Insert as child
)

// Insert a sibling block
await logseq.Editor.insertBlock(block.uuid, 'Sibling content', { sibling: true })

// Batch insert blocks
await logseq.Editor.insertBatchBlock(
  parentUuid,
  [
    { content: 'First item' },
    { content: 'Second item', children: [{ content: 'Nested item' }] },
    { content: 'Third item' },
  ],
  { sibling: true }
)

// Get block with full tree
const blockTree = await logseq.Editor.getBlock(block.uuid, {
  includeChildren: true,
})

// Update block content
await logseq.Editor.updateBlock(block.uuid, 'Updated content')

// Append to page
await logseq.Editor.appendBlockInPage('My Page', 'Last block')

// Prepend to page
await logseq.Editor.prependBlockInPage('My Page', 'First block')
```

### Navigating Between Blocks

```typescript
// Get siblings
const prevBlock = await logseq.Editor.getPreviousSiblingBlock(block.uuid)
const nextBlock = await logseq.Editor.getNextSiblingBlock(block.uuid)

// Get selected blocks (multi-select)
const selected = await logseq.Editor.getSelectedBlocks()
for (const block of selected) {
  console.log(block.content)
}
```

## Properties and Tags

### Defining Property Types (Critical!)

```typescript
// ⚠️ MUST define property types BEFORE using them!

// Define a text property
await logseq.Editor.upsertProperty('author', {
  type: 'string',
})

// Define a number property
await logseq.Editor.upsertProperty('year', {
  type: 'number',
})

// Define a property with choices
await logseq.Editor.upsertProperty('priority', {
  type: 'string',
  choices: ['high', 'medium', 'low'],
})

// Define a checkbox property
await logseq.Editor.upsertProperty('isRead', {
  type: 'checkbox',
})

// Define a date property
await logseq.Editor.upsertProperty('dueDate', {
  type: 'date',
})

// Define a node property with tag restriction
await logseq.Editor.upsertProperty('relatedBook', {
  type: 'node',
  // Only nodes tagged #Book will appear
})

// Now you can use these properties!
```

### Working with Properties

```typescript
// Get all properties in graph
const allProps = await logseq.Editor.getAllProperties()

// Get specific property definition
const statusProp = await logseq.Editor.getProperty('status')

// Get block's properties
const props = await logseq.Editor.getBlockProperties(block.uuid)

// Get specific property value
const status = await logseq.Editor.getBlockProperty(block.uuid, 'status')

// Set property value
await logseq.Editor.upsertBlockProperty(block.uuid, 'status', 'done')

// Remove property
await logseq.Editor.removeBlockProperty(block.uuid, 'status')

// Get page properties
const pageProps = await logseq.Editor.getPageProperties('page-name')
```

### Date Property Handling (Important!)

```typescript
// ❌ WRONG: Date properties don't accept strings!
await logseq.Editor.upsertBlockProperty(uuid, 'dueDate', '2025-01-15')

// ✅ CORRECT: Date properties store journal page IDs!

// Step 1: Create/get journal page
const journalPage = await logseq.Editor.createPage(
  '2025-01-15',
  {},
  {
    redirect: false,
  }
)

// Step 2: Set date property to journal page ID
await logseq.Editor.upsertBlockProperty(block.uuid, 'dueDate', journalPage.id)
// Note: using .id (number), not UUID (string)

// Helper function for date properties
async function setDateProperty(blockUuid: string, propName: string, dateStr: string) {
  const journal = await logseq.Editor.createPage(
    dateStr,
    {},
    {
      redirect: false,
    }
  )
  await logseq.Editor.upsertBlockProperty(blockUuid, propName, journal.id)
}

// Usage
await setDateProperty(block.uuid, 'deadline', '2025-01-15')
```

### Creating Tags (Classes)

```typescript
// Create a simple tag
const personTag = await logseq.Editor.createTag('Person')

// Create tag with custom UUID
const dataTag = await logseq.Editor.createTag('my-plugin-data', {
  uuid: 'my-unique-tag-uuid',
})

// Define properties for tag
await logseq.Editor.upsertProperty('lastName', { type: 'string' })
await logseq.Editor.upsertProperty('company', { type: 'string' })

// Add properties to tag schema
await logseq.Editor.addTagProperty(personTag.uuid, 'lastName')
await logseq.Editor.addTagProperty(personTag.uuid, 'company')

// Complete tag schema initialization pattern
async function initTagSchema() {
  // 1. Create tag
  const tag = await logseq.Editor.createTag('Book')

  // 2. Define property types
  const props = {
    author: 'string',
    year: 'number',
    rating: 'number',
    isRead: 'checkbox',
  }

  // 3. Add properties to tag
  for (const [name, type] of Object.entries(props)) {
    await logseq.Editor.upsertProperty(name, { type })
    await logseq.Editor.addTagProperty(tag.uuid, name)
  }
}
```

### Tag Inheritance

```typescript
// Create parent tag
const mediaTag = await logseq.Editor.createTag('MediaObject')
await logseq.Editor.upsertProperty('duration', { type: 'number' })
await logseq.Editor.addTagProperty(mediaTag.uuid, 'duration')

// Create child tag (inherits duration property)
const audioBookTag = await logseq.Editor.createTag('AudioBook')
// AudioBook now inherits 'duration' from MediaObject

// Add additional properties
await logseq.Editor.upsertProperty('narrator', { type: 'string' })
await logseq.Editor.addTagProperty(audioBookTag.uuid, 'narrator')
```

## Queries

### Simple DSL Queries

```typescript
// Find all tasks with status 'done'
const results = await logseq.DB.q('(property status done)')

// Find all books
const books = await logseq.DB.q('(tags Book)')

// Combined query
const highPriorityTasks = await logseq.DB.q('(and (tags Task) (property priority high))')
```

### Advanced Datascript Queries

```typescript
// Find all tasks (DB version)
const tasks = await logseq.DB.datascriptQuery(`
  [:find (pull ?b [:block/uuid :block/title])
   :where
   [?b :block/tags ?t]
   [?t :db/ident :logseq.class/Task]
   [?b :logseq.property/status ?s]]
`)

// Find blocks with property value
const highPriority = await logseq.DB.datascriptQuery(`
  [:find (pull ?b [*])
   :where
   [?b :logseq.property/priority ?p]
   [(= ?p :logseq.property.value/high)]]
`)

// Find journal pages (DB version)
const journals = await logseq.DB.datascriptQuery(`
  [:find (pull ?p [:block/title :block/uuid])
   :where
   [?p :block/tags :logseq.class/Journal]]
`)

// Search by title (was :block/content)
const matches = await logseq.DB.datascriptQuery(`
  [:find (pull ?b [*])
   :where
   [?b :block/title ?t]
   [(clojure.string/includes? ?t "search term")]]
`)

// Pages in namespace
const nsPages = await logseq.DB.datascriptQuery(`
  [:find (pull ?p [:block/title])
   :where
   [?p :block/title ?n]
   [(clojure.string/starts-with? ?n "project/")]]
`)
```

### Query with Inputs

```typescript
// Tasks due this week
const weeklyTasks = await logseq.DB.datascriptQuery(`
  [:find (pull ?b [*])
   :in $ ?start ?end
   :where
   [?b :logseq.property/deadline ?d]
   [(>= ?d ?start)]
   [(<= ?d ?end)]]
`, null, [:today, :7d_after]);
```

### Subscribe to Changes

```typescript
// Subscribe to all block changes
const unsub = logseq.DB.onChanged(({ blocks, txData }) => {
  console.log('Blocks changed:', blocks)
  blocks.forEach((block) => {
    console.log('Block:', block.title, 'UUID:', block.uuid)
  })
})

// Subscribe to specific block changes
const unsubBlock = logseq.DB.onBlockChanged(block.uuid, (block) => {
  console.log('Block updated:', block)
})

// Unsubscribe later
unsub()
unsubBlock()
```

## UI Integration

### Toast Messages

```typescript
// Show toast messages
await logseq.UI.showMsg('Success!', 'success')
await logseq.UI.showMsg('Error occurred', 'error')
await logseq.UI.showMsg('Info message', 'info')

// Toast with timeout and key
const key = await logseq.UI.showMsg('Loading...', 'info', {
  timeout: 5000,
})

// Close toast programmatically
logseq.UI.closeMsg(key)
```

### Slash Commands

```typescript
// Register a slash command
logseq.Editor.registerSlashCommand('My Command', async () => {
  const block = await logseq.Editor.getCurrentBlock()
  if (block) {
    await logseq.Editor.insertBlock(block.uuid, 'Command executed!')
  }
})

// Command that inserts content
logseq.Editor.registerSlashCommand('Insert Date', async () => {
  const block = await logseq.Editor.getCurrentBlock()
  if (block) {
    const today = new Date().toISOString().split('T')[0]
    await logseq.Editor.updateBlock(block.uuid, `[[${today}]]`)
  }
})
```

### Block Context Menu

```typescript
// Add item to block context menu
logseq.Editor.registerBlockContextMenuItem('Copy UUID', async ({ uuid }) => {
  await navigator.clipboard.writeText(uuid)
  logseq.UI.showMsg('UUID copied!', 'success')
})

// Context menu with more complex logic
logseq.Editor.registerBlockContextMenuItem('Convert to Task', async ({ uuid }) => {
  await logseq.Editor.upsertBlockProperty(uuid, 'Status', 'Todo')
  logseq.UI.showMsg('Converted to task', 'success')
})
```

### Keyboard Shortcuts

```typescript
// Register keyboard shortcut
logseq.App.registerCommandShortcut({ binding: 'mod+shift+t' }, async () => {
  logseq.UI.showMsg('Shortcut triggered!')
})

// Multiple shortcuts
logseq.App.registerCommandShortcut({ binding: 'mod+shift+p' }, async () => {
  // Open plugin settings
})
```

### Command Palette

```typescript
// Register command palette item
logseq.App.registerCommandPalette(
  {
    key: 'my-plugin-action',
    label: 'My Plugin: Do Something',
    keybinding: 'mod+shift+d',
  },
  async () => {
    logseq.UI.showMsg('Action executed!')
  }
)
```

### UI Injection

```typescript
// Add custom styles
logseq.provideStyle(`
  .my-plugin-highlight {
    background-color: yellow;
    padding: 2px 4px;
    border-radius: 3px;
  }
  .my-plugin-button {
    cursor: pointer;
    padding: 5px 10px;
  }
`)

// Inject UI element
logseq.provideUI({
  key: 'my-panel',
  path: '#app-container',
  template: `
    <div class="my-plugin-panel">
      <button data-on-click="handleClick">Click Me</button>
    </div>
  `,
})

// Add model for event handling
logseq.provideModel({
  handleClick() {
    logseq.UI.showMsg('Button clicked!')
  },
})
```

## Settings and Storage

### Settings Schema

```typescript
// Define settings
logseq.useSettingsSchema([
  {
    key: 'apiKey',
    type: 'string',
    default: '',
    title: 'API Key',
    description: 'Enter your API key',
  },
  {
    key: 'enabled',
    type: 'boolean',
    default: true,
    title: 'Enable Plugin',
    description: 'Turn plugin on/off',
  },
  {
    key: 'theme',
    type: 'enum',
    default: 'auto',
    enumChoices: ['light', 'dark', 'auto'],
    enumPicker: 'select',
    title: 'Theme',
    description: 'Choose theme preference',
  },
  {
    key: 'maxResults',
    type: 'number',
    default: 10,
    title: 'Max Results',
    description: 'Maximum number of results to show',
  },
])
```

### Using Settings

```typescript
// Access settings
const settings = logseq.settings
console.log('API Key:', settings.apiKey)
console.log('Enabled:', settings.enabled)

// Listen for settings changes
logseq.onSettingsChanged((newSettings, oldSettings) => {
  console.log('Settings changed:', oldSettings, '→', newSettings)
  if (newSettings.apiKey !== oldSettings.apiKey) {
    // Reinitialize with new API key
  }
})

// Update settings
logseq.updateSettings({ enabled: false })
```

### File Storage

```typescript
// Store data
const cacheData = {
  timestamp: Date.now(),
  data: [1, 2, 3],
}
await logseq.FileStorage.setItem('cache.json', JSON.stringify(cacheData))

// Retrieve data
const raw = await logseq.FileStorage.getItem('cache.json')
if (raw) {
  const parsed = JSON.parse(raw)
  console.log('Loaded cache:', parsed)
}

// Remove data
await logseq.FileStorage.removeItem('cache.json')

// List all keys
const keys = await logseq.FileStorage.listKeys()
console.log('Stored keys:', keys)
```

### HTTP Requests

```typescript
// Simple GET request
const data = await logseq.Request._request({
  url: 'https://api.example.com/data',
  method: 'GET',
  returnType: 'json',
})

// POST with headers
const result = await logseq.Request._request({
  url: 'https://api.example.com/create',
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    Authorization: `Bearer ${apiKey}`,
  },
  body: JSON.stringify({ name: 'Test' }),
  returnType: 'json',
})
```

## Event Handling

### App Events

```typescript
// Current graph changed
logseq.App.onCurrentGraphChanged(() => {
  console.log('Graph changed!')
})

// Theme changed
logseq.App.onThemeModeChanged(({ mode }) => {
  console.log('Theme:', mode) // 'light' or 'dark'
})

// Route changed
logseq.App.onRouteChanged(({ path }) => {
  console.log('Navigated to:', path)
})
```

### Cleanup on Unload

```typescript
// Register cleanup handler
logseq.beforeunload(async () => {
  // Save state
  await logseq.FileStorage.setItem(
    'state.json',
    JSON.stringify({
      lastUsed: Date.now(),
    })
  )

  // Close subscriptions
  unsub()

  console.log('Plugin unloading...')
})
```

## Complete Plugin Examples

### Book Import Plugin

```typescript
import '@logseq/libs'

async function main() {
  // Settings
  logseq.useSettingsSchema([{ key: 'apiKey', type: 'string', default: '', title: 'API Key' }])

  // Initialize Book tag schema
  await initBookSchema()

  // Register slash command
  logseq.Editor.registerSlashCommand('Import Book', async () => {
    await importBookDialog()
  })

  // Register command palette
  logseq.App.registerCommandPalette({ key: 'import-book', label: 'Import Book from API' }, importBookDialog)
}

async function initBookSchema() {
  // Create Book tag
  const bookTag = await logseq.Editor.createTag('Book')

  // Define properties
  const props = {
    author: 'string',
    year: 'number',
    rating: 'number',
    isbn: 'string',
    pageCount: 'number',
  }

  for (const [name, type] of Object.entries(props)) {
    await logseq.Editor.upsertProperty(name, { type })
    await logseq.Editor.addTagProperty(bookTag.uuid, name)
  }
}

async function importBookDialog() {
  const block = await logseq.Editor.getCurrentBlock()
  if (!block) {
    logseq.UI.showMsg('Please select a block first', 'error')
    return
  }

  // Get ISBN from block title
  const isbn = block.title.trim()

  // Fetch book data
  const book = await fetchBookData(isbn)

  if (book) {
    // Create page with tags and properties
    await logseq.Editor.createPage(book.title, {
      tags: ['Book'],
      author: book.author,
      year: book.year,
      rating: 0,
      isbn: book.isbn,
      pageCount: book.pages,
    })

    logseq.UI.showMsg(`Imported: ${book.title}`, 'success')
  }
}

async function fetchBookData(isbn: string) {
  // Implement API call here
  // Return { title, author, year, isbn, pages }
  return null
}

logseq.ready(main).catch(console.error)
```

### Task Timer Plugin

```typescript
import '@logseq/libs'

const timers = new Map<string, number>()

async function main() {
  // Register slash commands
  logseq.Editor.registerSlashCommand('Start Timer', async () => {
    const block = await logseq.Editor.getCurrentBlock()
    if (block) {
      startTimer(block.uuid)
    }
  })

  logseq.Editor.registerSlashCommand('Stop Timer', async () => {
    const block = await logseq.Editor.getCurrentBlock()
    if (block) {
      stopTimer(block.uuid)
    }
  })

  // Cleanup on unload
  logseq.beforeunload(() => {
    timers.forEach((_, uuid) => stopTimer(uuid))
  })
}

function startTimer(uuid: string) {
  if (timers.has(uuid)) {
    logseq.UI.showMsg('Timer already running', 'error')
    return
  }

  const startTime = Date.now()
  timers.set(uuid, startTime)

  logseq.UI.showMsg('Timer started!', 'success')

  // Update block every second
  const interval = setInterval(async () => {
    const elapsed = Date.now() - startTime
    const minutes = Math.floor(elapsed / 60000)
    const seconds = Math.floor((elapsed % 60000) / 1000)

    const block = await logseq.Editor.getBlock(uuid)
    if (block) {
      const content = block.title.replace(/\[\d+:\d+\]/, '')
      await logseq.Editor.updateBlock(uuid, `${content} [${minutes}:${seconds.toString().padStart(2, '0')}]`)
    }
  }, 1000)

  // Store interval ID for cleanup
  timers.set(`${uuid}-interval`, interval as unknown as number)
}

function stopTimer(uuid: string) {
  const startTime = timers.get(uuid)
  if (!startTime) {
    logseq.UI.showMsg('No timer running', 'error')
    return
  }

  const elapsed = Date.now() - startTime
  const minutes = Math.floor(elapsed / 60000)

  // Clear interval
  const interval = timers.get(`${uuid}-interval`)
  if (interval) {
    clearInterval(interval as unknown as NodeJS.Timeout)
  }

  timers.delete(uuid)
  timers.delete(`${uuid}-interval`)

  logseq.UI.showMsg(`Timer stopped: ${minutes} minutes`, 'info')
}

logseq.ready(main).catch(console.error)
```

### Query Builder Plugin

```typescript
import '@logseq/libs'

async function main() {
  // Register slash command to insert query
  logseq.Editor.registerSlashCommand('All Tasks Query', async () => {
    const block = await logseq.Editor.getCurrentBlock()
    if (block) {
      const query = `#+BEGIN_QUERY
{:title "All Tasks"
 :query [:find (pull ?b [*])
          :where
          [?b :block/tags ?t]
          [?t :db/ident :logseq.class/Task]]}
#+END_QUERY`

      await logseq.Editor.updateBlock(block.uuid, query)
    }
  })

  // Register command palette for custom query
  logseq.App.registerCommandPalette(
    {
      key: 'query-by-tag',
      label: 'Query by Tag',
      keybinding: 'mod+shift+q',
    },
    async () => {
      const tag = await promptUser('Enter tag name:')
      if (tag) {
        insertQueryByTag(tag)
      }
    }
  )
}

async function promptUser(message: string): Promise<string | null> {
  // Simple prompt implementation
  return window.prompt(message)
}

async function insertQueryByTag(tag: string) {
  const block = await logseq.Editor.getCurrentBlock()
  if (block) {
    const query = `#+BEGIN_QUERY
{:title "Items tagged ${tag}"
 :query [:find (pull ?b [*])
          :where
          [?b :block/tags ?t]
          [?t :block/title "${tag}"]]}
#+END_QUERY`

    await logseq.Editor.updateBlock(block.uuid, query)
  }
}

logseq.ready(main).catch(console.error)
```

## Testing Patterns

### Check Graph Type

```typescript
async function isDBGraph(): Promise<boolean> {
  const graph = await logseq.App.getCurrentGraph()
  // DB graphs have .sqlite extension
  return graph?.path?.endsWith('.sqlite') || false
}
```

### Safe Property Access

```typescript
// Helper to get property safely
async function getBlockPropertySafe(uuid: string, propName: string): Promise<any> {
  try {
    const props = await logseq.Editor.getBlockProperties(uuid)
    return props?.[propName]
  } catch (error) {
    console.error('Error getting property:', error)
    return null
  }
}
```

### Batch Operations

```typescript
// Process multiple blocks efficiently
async function batchProcessBlocks(blockUuids: string[], processor: (uuid: string) => Promise<void>) {
  const batchSize = 10

  for (let i = 0; i < blockUuids.length; i += batchSize) {
    const batch = blockUuids.slice(i, i + batchSize)
    await Promise.all(batch.map(processor))

    // Small delay between batches
    await new Promise((resolve) => setTimeout(resolve, 100))
  }
}
```

---

**Tips:**

- Always use `await` with `logseq.*` calls
- Define property types before using them
- Use number IDs for date properties, not strings
- Clean up subscriptions in `beforeunload`
- Test on both DB graphs and file graphs if supporting both
