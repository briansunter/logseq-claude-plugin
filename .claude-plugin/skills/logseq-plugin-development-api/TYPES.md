# @logseq/libs TypeScript Type Reference

Complete TypeScript type definitions for `@logseq/libs` (v0.0.11-beta.0). Extracted from the official npm package for DB
version plugin development.

## Table of Contents

- [Core Types](#core-types)
- [Plugin Types](#plugin-types)
- [Block and Page Types](#block-and-page-types)
- [Property Types](#property-types)
- [UI Types](#ui-types)
- [Event Types](#event-types)
- [Command Types](#command-types)
- [Storage Types](#storage-types)
- [Git Types](#git-types)
- [Database Types](#database-types)
- [Asset Types](#asset-types)

## Core Types

### Basic Identifiers

```typescript
// Entity ID (internal database ID)
type EntityID = number

// Block UUID (string format like "uuid-xxx-xxx")
type BlockUUID = string

// Block UUID tuple format
type BlockUUIDTuple = ['uuid', BlockUUID]

// Plugin local identity
type PluginLocalIdentity = string
```

### Theme Types

```typescript
// Theme mode
type ThemeMode = 'light' | 'dark'

// Legacy theme interface
interface LegacyTheme {
  name: string
  url: string
  description?: string
  mode?: ThemeMode
  pid: PluginLocalIdentity
}

// Theme (extends legacy)
interface Theme extends LegacyTheme {
  mode: ThemeMode
}

// Style string
type StyleString = string
```

## Plugin Types

### Plugin Configuration

```typescript
// Plugin package configuration from package.json
interface LSPluginPkgConfig {
  id: PluginLocalIdentity // Must be unique
  main: string // Entry point
  entry: string // Entry file
  title: string // Display name
  mode: 'shadow' | 'iframe' // Rendering mode
  themes: Theme[] // Plugin themes
  icon: string // Icon path
  devEntry?: unknown // Dev entrypoint
  theme?: unknown // Legacy theme
}

// Plugin base information
interface LSPluginBaseInfo {
  id: string // Must be unique
  mode: 'shadow' | 'iframe' // Rendering mode
  settings: {
    disabled: boolean
  } & Record<string, unknown>
  effect: boolean // Plugin effect mode
  iir?: boolean // Internal: installed in dot root
  lsr?: string // Internal: settings repo
}
```

### Plugin Instance

```typescript
// Main plugin user interface
interface ILSPluginUser extends EventEmitter<LSPluginUserEvents> {
  connected: boolean // Connection status
  caller: LSPluginCaller // RPC caller
  baseInfo: LSPluginBaseInfo // Plugin config
  settings?: LSPluginBaseInfo['settings'] // User settings

  // Lifecycle methods
  ready(model?: Record<string, any>): Promise<any>
  ready(callback?: (e: any) => void | {}): Promise<any>
  beforeunload(callback: () => Promise<void>): void

  // UI methods
  provideModel(model: Record<string, any>): this
  provideTheme(theme: Theme): this
  provideStyle(style: StyleString | StyleOptions): this
  provideUI(ui: UIOptions): this

  // Settings methods
  useSettingsSchema(schema: Array<SettingSchemaDesc>): this
  updateSettings(attrs: Record<string, any>): void
  onSettingsChanged<T>(cb: (a: T, b: T) => void): IUserOffHook
  showSettingsUI(): void
  hideSettingsUI(): void

  // Main UI control
  setMainUIAttrs(attrs: Partial<UIContainerAttrs>): void
  setMainUIInlineStyle(style: CSS.Properties): void
  hideMainUI(opts?: { restoreEditingCursor: boolean }): void
  showMainUI(opts?: { autoFocus: boolean }): void
  toggleMainUI(): void

  // Properties
  get version(): string
  get isMainUIVisible(): boolean
  get connected(): boolean
  get baseInfo(): LSPluginBaseInfo
  get effect(): Boolean
  get logger(): PluginLogger
  get settings(): {
    disabled: boolean
  } & Record<string, unknown>

  // API proxies
  get App(): IAppProxy
  get Editor(): IEditorProxy
  get DB(): IDBProxy
  get Git(): IGitProxy
  get UI(): IUIProxy
  get Assets(): IAssetsProxy
  get FileStorage(): LSPluginFileStorage
  get Request(): LSPluginRequest
  get Experiments(): LSPluginExperiments
}
```

## Block and Page Types

### Block Entity

```typescript
// Block - Logseq's fundamental data structure
interface BlockEntity {
  id: EntityID // Internal database ID
  uuid: BlockUUID // UUID for API calls
  left: IEntityID // Left sibling block
  format: 'markdown' | 'org' // Content format
  parent: IEntityID // Parent block/page
  content: string // Block content (use title in DB graphs)
  page: IEntityID // Parent page
  properties?: Record<string, any> // Block properties
  anchor?: string // Block anchor
  body?: any // Block body data
  children?: Array<BlockEntity | BlockUUIDTuple> // Child blocks
  container?: string // Container type
  file?: IEntityID // File reference
  level?: number // Indentation level
  meta?: {
    timestamps: any // Block timestamps
    properties: any // Meta properties
    startPos: number // Start position
    endPos: number // End position
  }
  title?: Array<any> // Block title (legacy)
  marker?: string // Task marker (file graph)
  [key: string]: unknown // Additional properties
}
```

### Page Entity

```typescript
// Page entity (extends block structure)
interface PageEntity {
  id: EntityID // Internal database ID
  uuid: BlockUUID // UUID for API calls
  name: string // Page name
  originalName: string // Original page name
  'journal?': boolean // Is journal page?
  file?: IEntityID // File reference
  namespace?: IEntityID // Parent namespace
  children?: Array<PageEntity> // Child pages
  properties?: Record<string, any> // Page properties
  format?: 'markdown' | 'org' // Content format
  journalDay?: number // Journal day (YYYYMMDD)
  updatedAt?: number // Last updated timestamp
  [key: string]: unknown // Additional properties
}
```

### Block and Page Identities

```typescript
// Block identity types
type BlockIdentity = BlockUUID | Pick<BlockEntity, 'uuid'>

// Page name
type BlockPageName = string

// Page identity (can be name or UUID)
type PageIdentity = BlockPageName | BlockIdentity

// Entity ID wrapper
type IEntityID = {
  id: EntityID
  [key: string]: any
}
```

### Batch Operations

```typescript
// Batch block structure
interface IBatchBlock {
  content: string
  properties?: Record<string, any>
  children?: Array<IBatchBlock>
}
```

## Property Types

### Property Values (DB Version)

```typescript
// Property types (DB graphs only)
type PropertyType =
  | 'string' // Text content (default)
  | 'number' // Numeric values (integers, floats)
  | 'date' // Date (journal page Entity ID)
  | 'datetime' // DateTime (timestamp milliseconds)
  | 'checkbox' // Boolean (true/false)
  | 'url' // URL strings only
  | 'node' // References to other nodes (pages/blocks)
```

### Property Operations

```typescript
// Setting schema description
interface SettingSchemaDesc {
  key: string
  type: 'string' | 'number' | 'boolean' | 'enum' | 'object' | 'heading'
  default: string | number | boolean | Array<any> | object | null
  title: string
  description: string
  inputAs?: 'color' | 'date' | 'datetime-local' | 'range' | 'textarea'
  enumChoices?: Array<string>
  enumPicker?: 'select' | 'radio' | 'checkbox'
}

// Block property
type BlockProperty = {
  type: PropertyType
  choices?: Array<{ value: any; label?: string; icon?: string }>
  multiple?: boolean
  ui?: {
    position?: 'block-top' | 'block-bottom' | 'block-right' | 'block-end'
    hideWhenEmpty?: boolean
    hideByDefault?: boolean
  }
  defaultValue?: any
}
```

## UI Types

### UI Container Options

```typescript
// UI container attributes
type UIContainerAttrs = {
  draggable: boolean
  resizable: boolean
}

// Base UI options
type UIBaseOptions = {
  key?: string // Unique identifier
  replace?: boolean // Replace existing UI
  template: string | null // HTML template
  style?: CSS.Properties // CSS styles
  attrs?: Record<string, string> // HTML attributes
  close?: 'outside' | string // Close behavior
  reset?: boolean // Reset state
}

// UI path identity (DOM selector)
type UIPathIdentity = {
  path: string // DOM selector
}

// UI slot identity (slot key)
type UISlotIdentity = {
  slot: string // Slot key
}

// UI slot options
type UISlotOptions = UIBaseOptions & UISlotIdentity

// UI path options
type UIPathOptions = UIBaseOptions & UIPathIdentity

// Combined UI options
type UIOptions = UIBaseOptions | UIPathOptions | UISlotOptions
```

### Message Options

```typescript
// UI message options
type UIMsgOptions = {
  key: string // Message key for closing
  timeout: number // Auto-close timeout
}

// Message key type
type UIMsgKey = UIMsgOptions['key']
```

## Event Types

### Hooks and Events

```typescript
// Generic hook event
type IHookEvent = {
  [key: string]: any
}

// User off-hook (returns cleanup function)
type IUserOffHook = () => void

// User hook
type IUserHook<E = any, R = IUserOffHook> = (callback: (e: IHookEvent & E) => void) => IUserOffHook

// User slot hook
type IUserSlotHook<E = any> = (callback: (e: IHookEvent & UISlotIdentity & E) => void) => void

// User condition slot hook
type IUserConditionSlotHook<C = any, E = any> = (
  condition: C,
  callback: (e: IHookEvent & UISlotIdentity & E) => void
) => void
```

### Plugin Events

```typescript
// Plugin user events
type LSPluginUserEvents = 'ui:visible:changed' | 'settings:changed'
```

## Command Types

### Slash Commands

```typescript
// Slash command actions
type SlashCommandActionCmd =
  | 'editor/input'
  | 'editor/hook'
  | 'editor/clear-current-slash'
  | 'editor/restore-saved-cursor'

// Slash command action tuple
type SlashCommandAction = [cmd: SlashCommandActionCmd, ...args: any]

// Simple command callback
type SimpleCommandCallback<E = any> = (e: IHookEvent & E) => void

// Block command callback
type BlockCommandCallback = (
  e: IHookEvent & {
    uuid: BlockUUID
  }
) => Promise<void>
```

### Keybinding Types

```typescript
// Keybinding
type Keybinding = string | Array<string>

// Simple command keybinding
type SimpleCommandKeybinding = {
  mode?: 'global' | 'non-editing' | 'editing'
  binding: Keybinding
  mac?: string // macOS-specific binding
}
```

### External Commands

```typescript
// External command types
type ExternalCommandType =
  | 'logseq.command/run'
  | 'logseq.editor/cycle-todo'
  | 'logseq.editor/down'
  | 'logseq.editor/up'
  | 'logseq.editor/expand-block-children'
  | 'logseq.editor/collapse-block-children'
  | 'logseq.editor/open-file-in-default-app'
  | 'logseq.editor/open-file-in-directory'
  | 'logseq.editor/select-all-blocks'
  | 'logseq.editor/toggle-open-blocks'
  | 'logseq.editor/zoom-in'
  | 'logseq.editor/zoom-out'
  | 'logseq.editor/indent'
  | 'logseq.editor/outdent'
  | 'logseq.editor/copy'
  | 'logseq.editor/cut'
  | 'logseq.go/home'
  | 'logseq.go/journals'
  | 'logseq.go/keyboard-shortcuts'
  | 'logseq.go/next-journal'
  | 'logseq.go/prev-journal'
  | 'logseq.go/search'
  | 'logseq.go/tomorrow'
  | 'logseq.go/backward'
  | 'logseq.go/forward'
  | 'logseq.search/re-index'
  | 'logseq.sidebar/clear'
  | 'logseq.sidebar/open-today-page'
  | 'logseq.ui/goto-plugins'
  | 'logseq.ui/select-theme-color'
  | 'logseq.ui/toggle-brackets'
  | 'logseq.ui/toggle-contents'
  | 'logseq.ui/toggle-document-mode'
  | 'logseq.ui/toggle-help'
  | 'logseq.ui/toggle-left-sidebar'
  | 'logseq.ui/toggle-right-sidebar'
  | 'logseq.ui/toggle-settings'
  | 'logseq.ui/toggle-wide-mode'
```

### Cursor Position

```typescript
// Block cursor position
type BlockCursorPosition = {
  left: number
  top: number
  height: number
  pos: number
  rect: DOMRect
}
```

## Storage Types

### File Storage

```typescript
// Async storage interface
interface IAsyncStorage {
  getItem(key: string): Promise<string | null>
  setItem(key: string, value: string): Promise<void>
  removeItem(key: string): Promise<void>
  keys(): Promise<string[]>
  clear(): Promise<void>
}

// LSPlugin file storage
interface LSPluginFileStorage {
  getItem(key: string): Promise<string | null>
  setItem(key: string, value: string): Promise<void>
  removeItem(key: string): Promise<void>
  keys(): Promise<string[]>
  clear(): Promise<void>
}
```

## Database Types

### Datom Operations

```typescript
// Datom (database atom)
type IDatom = [e: number, a: string, v: any, t: number, added: boolean]

// Database change event
interface DBChangedEvent {
  blocks: Array<BlockEntity>
  txData: Array<IDatom>
  txMeta?: {
    outlinerOp: string
    [key: string]: any
  }
}

// Database proxy interface
interface IDBProxy {
  // Run a DSL query
  q<T = any>(dsl: string): Promise<Array<T> | null>

  // Run a datascript query
  datascriptQuery<T = any>(query: string, ...inputs: Array<any>): Promise<T>

  // Hook all transaction data
  onChanged: IUserHook<DBChangedEvent>

  // Subscribe to specific block changes
  onBlockChanged(
    uuid: BlockUUID,
    callback: (
      block: BlockEntity,
      txData: Array<IDatom>,
      txMeta?: {
        outlinerOp: string
        [key: string]: any
      }
    ) => void
  ): IUserOffHook
}
```

### Query Types

```typescript
// Simple query DSL
type SimpleQueryDSL = string // "(property status done)"

// Datascript query
type DatascriptQuery = string // "[:find (pull ?b [*]) :where ...]"

// Query inputs
type QueryInputs = any[]
```

## Git Types

### Git Operations

```typescript
// Git command result
interface IGitResult {
  stdout: string
  stderr: string
  exitCode: number
}

// Git proxy interface
interface IGitProxy {
  // Execute git command (dugite API)
  execCommand(args: string[]): Promise<IGitResult>

  // Load .gitignore file
  loadIgnoreFile(): Promise<string>

  // Save .gitignore file
  saveIgnoreFile(content: string): Promise<void>
}
```

## Asset Types

### Asset Operations

```typescript
// Asset file info
interface AssetFileInfo {
  path: string
  size: number
  accessTime: number
  modifiedTime: number
  changeTime: number
  birthTime: number
}

// Assets proxy interface
interface IAssetsProxy {
  // List files in current graph
  listFilesOfCurrentGraph(exts?: string | string[]): Promise<Array<AssetFileInfo>>

  // Create sandbox storage
  makeSandboxStorage(): IAsyncStorage

  // Make asset URL
  makeUrl(path: string): Promise<string>

  // Open asset in Logseq
  builtInOpen(path: string): Promise<boolean | undefined>
}
```

## App Types

### App Information

```typescript
// App information
interface AppInfo {
  version: string
  [key: string]: unknown
}

// App user info
interface AppUserInfo {
  [key: string]: any
}

// User configurations
interface AppUserConfigs {
  preferredThemeMode: ThemeMode
  preferredFormat: 'markdown' | 'org'
  preferredDateFormat: string
  preferredStartOfWeek: string
  preferredLanguage: string
  preferredWorkflow: string
  currentGraph: string
  showBracket: boolean
  enabledFlashcards: boolean
  enabledJournals: boolean
  [key: string]: unknown
}
```

### Graph Information

```typescript
// Graph information
interface AppGraphInfo {
  name: string
  url: string
  path: string
  [key: string]: unknown
}
```

### App Proxy Interface

```typescript
// App proxy interface
interface IAppProxy {
  // Get app info
  getInfo(key?: keyof AppInfo): Promise<AppInfo | any>
  getUserInfo(): Promise<AppUserInfo | null>
  getUserConfigs(): Promise<AppUserConfigs>

  // Current graph
  getCurrentGraph(): Promise<AppGraphInfo | null>
  getCurrentGraphConfigs(...keys: string[]): Promise<any>
  setCurrentGraphConfigs(configs: {}): Promise<void>
  getCurrentGraphFavorites(): Promise<Array<string> | null>
  getCurrentGraphRecent(): Promise<Array<string> | null>
  getCurrentGraphTemplates(): Promise<Record<string, BlockEntity> | null>

  // Templates
  getTemplate(name: string): Promise<BlockEntity | null>
  existTemplate(name: string): Promise<Boolean>
  createTemplate(
    target: BlockUUID,
    name: string,
    opts?: {
      overwrite: boolean
    }
  ): Promise<any>
  removeTemplate(name: string): Promise<any>
  insertTemplate(target: BlockUUID, name: string): Promise<any>

  // Navigation
  pushState(k: string, params?: Record<string, any>, query?: Record<string, any>): void
  replaceState(k: string, params?: Record<string, any>, query?: Record<string, any>): void

  // UI control
  setZoomFactor(factor: number): void
  setFullScreen(flag: boolean | 'toggle'): void
  setLeftSidebarVisible(flag: boolean | 'toggle'): void
  setRightSidebarVisible(flag: boolean | 'toggle'): void
  clearRightSidebarBlocks(opts?: { close: boolean }): void

  // Register commands and UI items
  registerCommand(
    type: string,
    opts: {
      key: string
      label: string
      desc?: string
      palette?: boolean
      keybinding?: SimpleCommandKeybinding
    },
    action: SimpleCommandCallback
  ): void

  registerCommandPalette(
    opts: {
      key: string
      label: string
      keybinding?: SimpleCommandKeybinding
    },
    action: SimpleCommandCallback
  ): void

  registerCommandShortcut(
    keybinding: SimpleCommandKeybinding | string,
    action: SimpleCommandCallback,
    opts?: Partial<{
      key: string
      label: string
      desc: string
      extras: Record<string, any>
    }>
  ): void

  registerUIItem(
    type: 'toolbar' | 'pagebar',
    opts: {
      key: string
      template: string
    }
  ): void

  registerPageMenuItem(tag: string, action: (e: IHookEvent & { page: string }) => void): void

  registerSearchService<T extends IPluginSearchServiceHooks>(s: T): void

  // External commands
  invokeExternalCommand(type: ExternalCommandType, ...args: Array<any>): Promise<void>
  invokeExternalPlugin(type: string, ...args: Array<any>): Promise<unknown>
  getExternalPlugin(pid: string): Promise<{} | null>

  // App state
  getStateFromStore<T = any>(path: string | Array<string>): Promise<T>
  setStateFromStore(path: string | Array<string>, value: any): Promise<void>

  // App lifecycle
  relaunch(): Promise<void>
  quit(): Promise<void>

  // Events
  onCurrentGraphChanged: IUserHook
  onGraphAfterIndexed: IUserHook<{ repo: string }>
  onThemeModeChanged: IUserHook<{ mode: 'dark' | 'light' }>
  onThemeChanged: IUserHook<
    Partial<{
      name: string
      mode: string
      pid: string
      url: string
    }>
  >
  onTodayJournalCreated: IUserHook<{ title: string }>

  onBeforeCommandInvoked(condition: ExternalCommandType | string, callback: (e: IHookEvent) => void): IUserOffHook

  onAfterCommandInvoked(condition: ExternalCommandType | string, callback: (e: IHookEvent) => void): IUserOffHook

  onBlockRendererSlotted: IUserConditionSlotHook<BlockUUID, Omit<BlockEntity, 'children' | 'page'>>

  onMacroRendererSlotted: IUserSlotHook<{
    payload: {
      arguments: Array<string>
      uuid: string
      [key: string]: any
    }
  }>

  onPageHeadActionsSlotted: IUserSlotHook
  onRouteChanged: IUserHook<{
    path: string
    template: string
  }>
  onSidebarVisibleChanged: IUserHook<{
    visible: boolean
  }>
}
```

## Editor Types

### Editor Proxy Interface

```typescript
// Editor proxy interface
interface IEditorProxy extends Record<string, any> {
  // Commands
  registerSlashCommand(tag: string, action: BlockCommandCallback | Array<SlashCommandAction>): unknown

  registerBlockContextMenuItem(label: string, action: BlockCommandCallback): unknown

  registerHighlightContextMenuItem(
    label: string,
    action: SimpleCommandCallback,
    opts?: { clearSelection: boolean }
  ): unknown

  // Editing state
  checkEditing(): Promise<BlockUUID | boolean>
  insertAtEditingCursor(content: string): Promise<void>
  restoreEditingCursor(): Promise<void>
  exitEditingMode(selectBlock?: boolean): Promise<void>
  getEditingCursorPosition(): Promise<BlockCursorPosition | null>
  getEditingBlockContent(): Promise<string>

  // Current context
  getCurrentPage(): Promise<PageEntity | BlockEntity | null>
  getCurrentBlock(): Promise<BlockEntity | null>
  getSelectedBlocks(): Promise<Array<BlockEntity> | null>
  getCurrentPageBlocksTree(): Promise<Array<BlockEntity>>
  getPageBlocksTree(srcPage: PageIdentity): Promise<Array<BlockEntity>>
  getPageLinkedReferences(srcPage: PageIdentity): Promise<Array<[page: PageEntity, blocks: Array<BlockEntity>]> | null>

  // Namespaces
  getPagesFromNamespace(namespace: BlockPageName): Promise<Array<PageEntity> | null>
  getPagesTreeFromNamespace(namespace: BlockPageName): Promise<Array<PageEntity> | null>

  // Block operations
  newBlockUUID(): Promise<string>
  insertBlock(
    srcBlock: BlockIdentity,
    content: string,
    opts?: Partial<{
      before: boolean
      sibling: boolean
      isPageBlock: boolean
      focus: boolean
      customUUID: string
      properties: {}
    }>
  ): Promise<BlockEntity | null>

  insertBatchBlock(
    srcBlock: BlockIdentity,
    batch: IBatchBlock | Array<IBatchBlock>,
    opts?: Partial<{
      before: boolean
      sibling: boolean
      keepUUID: boolean
    }>
  ): Promise<Array<BlockEntity> | null>

  updateBlock(srcBlock: BlockIdentity, content: string, opts?: Partial<{ properties: {} }>): Promise<void>

  removeBlock(srcBlock: BlockIdentity): Promise<void>
  getBlock(
    srcBlock: BlockIdentity | EntityID,
    opts?: Partial<{ includeChildren: boolean }>
  ): Promise<BlockEntity | null>

  setBlockCollapsed(
    uuid: BlockUUID,
    opts:
      | {
          flag: boolean | 'toggle'
        }
      | boolean
      | 'toggle'
  ): Promise<void>

  // Page operations
  getPage(srcPage: PageIdentity | EntityID, opts?: Partial<{ includeChildren: boolean }>): Promise<PageEntity | null>

  createPage(
    pageName: BlockPageName,
    properties?: {},
    opts?: Partial<{
      redirect: boolean
      createFirstBlock: boolean
      format: BlockEntity['format']
      journal: boolean
    }>
  ): Promise<PageEntity | null>

  deletePage(pageName: BlockPageName): Promise<void>
  renamePage(oldName: string, newName: string): Promise<void>
  getAllPages(repo?: string): Promise<PageEntity[] | null>

  prependBlockInPage(
    page: PageIdentity,
    content: string,
    opts?: Partial<{ properties: {} }>
  ): Promise<BlockEntity | null>

  appendBlockInPage(
    page: PageIdentity,
    content: string,
    opts?: Partial<{ properties: {} }>
  ): Promise<BlockEntity | null>

  // Block navigation
  getPreviousSiblingBlock(srcBlock: BlockIdentity): Promise<BlockEntity | null>
  getNextSiblingBlock(srcBlock: BlockIdentity): Promise<BlockEntity | null>

  moveBlock(
    srcBlock: BlockIdentity,
    targetBlock: BlockIdentity,
    opts?: Partial<{
      before: boolean
      children: boolean
    }>
  ): Promise<void>

  editBlock(srcBlock: BlockIdentity, opts?: { pos: number }): Promise<void>

  selectBlock(srcBlock: BlockIdentity): Promise<void>

  saveFocusedCodeEditorContent(): Promise<void>

  // Properties
  upsertBlockProperty(block: BlockIdentity, key: string, value: any): Promise<void>

  removeBlockProperty(block: BlockIdentity, key: string): Promise<void>
  getBlockProperty(block: BlockIdentity, key: string): Promise<any>

  getBlockProperties(block: BlockIdentity): Promise<any>

  // Scroll and sidebar
  scrollToBlockInPage(pageName: BlockPageName, blockId: BlockIdentity, opts?: { replaceState: boolean }): void

  openInRightSidebar(id: BlockUUID | EntityID): void

  // Events
  onInputSelectionEnd: IUserHook<{
    caret: any
    point: { x: number; y: number }
    start: number
    end: number
    text: string
  }>
}
```

## Search Types

### Search Service

```typescript
// Search block item
type SearchBlockItem = {
  id: EntityID
  uuid: BlockIdentity
  content: string
  page: EntityID
}

// Search page item
type SearchPageItem = string

// Search file item
type SearchFileItem = string

// Search index status
type SearchIndiceInitStatus = boolean

// Plugin search service hooks
interface IPluginSearchServiceHooks {
  name: string
  options?: Record<string, any>

  onQuery(
    graph: string,
    key: string,
    opts?: Partial<{ limit: number }>
  ): Promise<{
    graph: string
    key: string
    blocks?: Array<Partial<SearchBlockItem>>
    pages?: Array<SearchPageItem>
    files?: Array<SearchFileItem>
  }>

  onIndiceInit(graph: string): Promise<SearchIndiceInitStatus>
  onIndiceReset(graph: string): Promise<void>

  onBlocksChanged(
    graph: string,
    changes: {
      added: Array<SearchBlockItem>
      removed: Array<EntityID>
    }
  ): Promise<void>

  onGraphRemoved(graph: string, opts?: {}): Promise<any>
}
```

## UI Proxy Interface

```typescript
// UI proxy interface
interface IUIProxy {
  // Messages
  showMsg(
    content: string,
    status?: 'success' | 'warning' | 'error' | string,
    opts?: Partial<UIMsgOptions>
  ): Promise<UIMsgKey>

  closeMsg(key: UIMsgKey): void

  // Element queries
  queryElementRect(selector: string): Promise<DOMRectReadOnly | null>
  queryElementById(id: string): Promise<string | boolean>

  // Slot validation
  checkSlotValid(slot: UISlotIdentity['slot']): Promise<boolean>

  // Theme CSS properties
  resolveThemeCssPropsVals(props: string | Array<string>): Promise<Record<string, string | undefined | null>>
}
```

---

## DB Version Type Updates

### Important Type Changes (File Graph → DB Graph)

| File Graph Type          | DB Graph Type                             | Notes                    |
| ------------------------ | ----------------------------------------- | ------------------------ |
| `BlockEntity.content`    | `BlockEntity.title`                       | Use `title` in DB graphs |
| `BlockEntity.marker`     | `:logseq.property/status`                 | Query via property       |
| `BlockEntity.priority`   | `:logseq.property/priority`               | Query via property       |
| `BlockEntity.deadline`   | `:logseq.property/deadline`               | Query via property       |
| `PageEntity['journal?']` | Query `:block/tags :logseq.class/Journal` | Use tag query            |
| `:block/content`         | `:block/title`                            | Attribute rename         |
| `:block/marker`          | `:logseq.property/status`                 | Attribute rename         |
| `:block/path-refs`       | Use `(has-ref ?b ?ref)` rule              | No longer supported      |

### New DB Graph Types

```typescript
// DB version specific types
interface DBBlockEntity extends BlockEntity {
  title: string // Was `content` in file graphs
  createdAt: number // Built-in timestamp
  updatedAt: number // Built-in timestamp
  tags?: TagEntity[] // New tags (classes)
}

interface DBPageEntity extends PageEntity {
  title: string // Was `name`/`originalName`
  type: 'page' | 'journal' | 'whiteboard' | 'class' | 'property' | 'hidden'
  tags?: TagEntity[] // New tags (classes)
  createdAt: number // Built-in timestamp
  updatedAt: number // Built-in timestamp
  journalDay?: number // YYYYMMDD format
}

// New tag (class) entity
interface TagEntity {
  id: EntityID
  uuid: BlockUUID
  title: string // Tag name
  extends?: TagEntity[] // Parent tags
  tagProperties?: PropertyEntity[] // Inherited properties
}
```

### Property Type Definitions (DB Graph)

```typescript
// Property definition (from API)
interface PropertyEntity {
  name: string
  type: PropertyType
  choices?: Array<{ value: any; label?: string; icon?: string }>
  multiple?: boolean
  defaultValue?: any
  ui?: {
    position?: 'block-top' | 'block-bottom' | 'block-right' | 'block-end'
    hideWhenEmpty?: boolean
    hideByDefault?: boolean
  }
}

// Typed property values
type PropertyValue =
  | string // 'string' type
  | number // 'number' type
  | EntityID // 'date' type (journal page ID)
  | number // 'datetime' type (timestamp)
  | boolean // 'checkbox' type
  | string // 'url' type
  | EntityID // 'node' type (page/block ID)
```

---

**Source:** Extracted from `@logseq/libs@0.0.11-beta.0` package

**Related Documentation:**

- SKILL.md - Plugin development guide
- REFERENCE.md - Complete API reference
- EXAMPLES.md - Code examples using these types

**TypeScript Setup:**

```json
// package.json
{
  "devDependencies": {
    "@logseq/libs": "latest"
  }
}
```

```typescript
// index.ts
import '@logseq/libs'

// Now logseq is fully typed!
const page: PageEntity | null = await logseq.Editor.getCurrentPage()
```
