# Logseq DB Version Examples

This file contains practical examples for working with Logseq DB graphs. See SKILL.md for conceptual documentation.

## Table of Contents

- [Tag and Property Examples](#tag-and-property-examples)
- [Task Management Examples](#task-management-examples)
- [Query Examples](#query-examples)
- [Template Examples](#template-examples)
- [Organization Examples](#organization-examples)

## Tag and Property Examples

### Creating a Person Tag with Properties

```markdown
# Define tag

Open Search → type #Person → Enter Add properties:

- lastName (Text)
- birthday (Date)
- company (Text)
- role (Text)

# Use the tag

- John Doe #Person lastName:: Smith birthday:: [1990-05-15] company:: Acme Corp role:: Engineer
```

### Creating an Inheritance Hierarchy

```markdown
# Base tag

#Book Tag Properties:

- author (Text)
- year (Number)
- rating (Number)

# Child tag (inherits properties)

#AudioBook extends:: #Book Tag Properties:

- narrator (Text)
- duration (Number)

# AudioBook automatically has all 5 properties

- The Great Gatsby (Audiobook) #AudioBook author:: F. Scott Fitzgerald year:: 1925 rating:: 9 narrator:: Jake Gyllenhaal
  duration:: 180
```

### Using Node Property Type

```markdown
# Define property with Node type

Property: author Type: Node Specify node tags: #Person

# Now author property only shows nodes tagged #Person

- My Book #Book author:: [[John Doe]] # Only Person-tagged nodes appear
```

## Task Management Examples

### Basic Task Creation

```markdown
# Method 1: Use slash command

- Write documentation /todo

# Method 2: Add Status property

- Write documentation Status:: Todo

# Method 3: End with #Task

- Write documentation #Task
```

### Task with All Properties

```markdown
- Complete project proposal #Task Status:: Doing Priority:: high Deadline:: 2025-01-15 Scheduled:: 2025-01-08
```

### Creating Custom Task Types

```markdown
# Create child tag of Task

#ProjectTask extends:: #Task Tag Properties:

- project (Node, specify #Project)
- assignedTo (Node, specify #Person)

# Use custom task type

- Fix authentication bug #ProjectTask Status:: Todo Priority:: high project:: [[Mobile App]] assignedTo:: [[John Doe]]
```

### Repeated Tasks

```markdown
# Daily standup meeting

- Daily standup #Task Status:: Todo Scheduled:: <2025-01-08 Mon 9:00>
  # In date picker, check "Repeat task"
  # Set interval: 1 Day

# When marked Done, automatically:

# - Status resets to Todo

# - Scheduled advances to next day (9:00)
```

## Query Examples

### Simple Queries

```clojure
# All TODO tasks
(property status todo)

# All high priority tasks
(and (tags Task) (property priority high))

# Tasks due this week
(between :today :7d-after)

# All books with rating > 8
(and (tags Book) (property rating > 8))

# Untagged pages
(not (tags))
```

### Advanced Queries

```clojure
# All incomplete tasks
#+BEGIN_QUERY
{:title "📋 All TODOs"
 :query [:find (pull ?b [*])
         :where
         [?b :block/tags ?t]
         [?t :db/ident :logseq.class/Task]
         [?b :logseq.property/status ?s]
         [(not= ?s :logseq.property.value/done)]]}
#+END_QUERY
```

```clojure
# Books grouped by author
#+BEGIN_QUERY
{:title "📚 Books by Author"
 :query [:find (pull ?b [:block/title])
         :where
         [?b :block/tags ?t]
         [?t :block/title "Book"]
         [?b :block/properties ?p]
         [(get ?p :author) ?author]]}
#+END_QUERY
```

```clojure
# Tasks due in next 7 days
#+BEGIN_QUERY
{:title "📅 Due This Week"
 :query [:find (pull ?b [*])
         :in $ ?start ?end
         :where
         [?b :logseq.property/deadline ?d]
         [(>= ?d ?start)]
         [(<= ?d ?end)]]
 :inputs [:today :7d-after]}
#+END_QUERY
```

```clojure
# Journal pages from current month
#+BEGIN_QUERY
{:title "📝 This Month's Journals"
 :query [:find (pull ?p [:block/title :block/uuid])
         :where
         [?p :block/tags :logseq.class/Journal]
         [?p :block/journalDay ?d]
         [(>= ?d 20250101)]
         [(<= ?d 20250131)]]}
#+END_QUERY
```

```clojure
# Find blocks referencing multiple tags
#+BEGIN_QUERY
{:title "🔗 Cross-Referenced"
 :query [:find (pull ?b [*])
         :where
         [?b :block/tags ?t1]
         [?b :block/tags ?t2]
         [?t1 :block/title "Book"]
         [?t2 :block/title "Favorite"]]}
#+END_QUERY
```

## Template Examples

### Meeting Notes Template

```markdown
- Meeting Notes #Template Apply template to tags:: #Meeting
  - **Date:** [[Today]]
  - ## **Attendees:**
  - ## **Agenda:**
  - ## **Discussion:**
  - **Action Items:**
    - /todo
  - **Next Meeting:** Scheduled:: <date picker>
```

### Book Notes Template

```markdown
- Book Notes #Template Apply template to tags:: #Book
  - **Title:** [[Book Title]]
  - **Author:** [[Author Name]]
  - **ISBN:**
  - ## **Summary:**
  - ## **Key Insights:**
    -
    -
  - **Quotes:**
    - > Quote 1
    - > Quote 2
  - ## **Related:**
```

### Project Template

```markdown
- Project Template #Template Apply template to tags:: #Project
  - **Project:** [[Project Name]]
  - **Status:** Backlog
  - **Start Date:** [[Today]]
  - **Target Date:** Deadline::
  - ## **Team Members:**
  - ## **Goals:**
  - **Milestones:**
    - /todo
    - /todo
    - /todo
  - ## **Resources:**
```

### Weekly Review Template

```markdown
- Weekly Review #Template Apply template to tags:: #Journal
  - ## Wins This Week
    -
    -
  - ## Challenges
    -
  - ## Lessons Learned
    -
  - ## Next Week Focus
    - /todo
    - /todo
  - ## Personal Reflection
    -
```

## Organization Examples

### PARA Method with Tags

```markdown
# Projects

#Project Tag Properties:

- status (Choices: Active, Archived, On Hold)
- goal (Text)
- deadline (Date)

# Areas

#Area Tag Properties:

- projects (Node, specify #Project)

# Resources

#Resource Tag Properties:

- type (Choices: Book, Article, Video, Course)
- status (Choices: To Read, Reading, Completed)
- rating (Number)

# Archive

#Archive Tag Properties:

- archivedDate (Date)
```

### Zettelkasten System

```markdown
# Permanent Notes

#Permanent Tag Properties:

- related (Node, multiple values)
- category (Choices: Concept, Method, Theory, Principle)

# Literature Notes

#Literature Tag Properties:

- source (Node, specify #Book, #Article)
- author (Text)
- page (Text)

# Fleeting Notes

#Fleeting Tag Properties:

- harvestDate (Date)
- convertedTo (Node, specify #Permanent)
```

### Knowledge Management Tags

```markdown
# Concept Tag

#Concept Tag Properties:

- definition (Text)
- examples (Text, multiple values)
- relatedConcepts (Node, multiple values)

# How-To Tag

#HowTo Tag Properties:

- difficulty (Choices: Beginner, Intermediate, Advanced)
- timeRequired (Text)
- prerequisites (Node, multiple values)

# Reference Tag

#Reference Tag Properties:

- type (Choices: API, Documentation, Tutorial, Guide)
- url (URL)
- lastAccessed (Date)
```

### GTD Workflow

```markdown
# Inbox (capture everything)

#Inbox

- [Idea, task, note] #Inbox

# Next Actions (context-based)

#NextAction Tag Properties:

- context (Choices: @home, @work, @phone, @computer, @errand)
- energy (Choices: High, Medium, Low)
- time (Choices: <5min, <15min, <30min, >30min)

# Waiting For

#Waiting Tag Properties:

- waitingFor (Node, specify #Person)
- followUpDate (Date)

# Someday/Maybe

#Someday Tag Properties:

- somedayPriority (Choices: Dream, Maybe, Later)
```

## Property Configuration Examples

### Status Property with Choices

```markdown
# Configure Status property

Property: Status Type: Text Choices:

- Backlog (icon: 📋)
- Todo (icon: ⬜)
- Doing (icon: 🔄)
- In Review (icon: 👀)
- Done (icon: ✅)
- Canceled (icon: ❌) Checkbox mapping: Done = checked, Todo = unchecked
```

### Priority Property

```markdown
Property: Priority Type: Text Choices:

- high (icon: 🔴, description: Urgent and important)
- medium (icon: 🟡, description: Important but not urgent)
- low (icon: 🟢, description: Nice to have)
```

### Multi-value Property

```markdown
Property: tags Type: Text Multiple values: true UI position: End of block
```

### Date Property with Repeater

```markdown
Property: reviewDate Type: Date Default value: :today UI position: Under block

# Usage in blocks:

- Review this #Card reviewDate:: [2025-01-15]
  # Date picker shows "Repeat task" option
```

## Advanced Patterns

### Cross-referencing System

```markdown
# Source material

- Original Idea #Source content:: "Raw idea text here"

# Notes referencing source

- Developed Note #Note basedOn:: [[Original Idea]] extends:: true

# Output creation

- Final Output #Output sources:: [[Original Idea]], [[Developed Note]]
```

### Progress Tracking

```markdown
# Milestone tracking

#Milestone Tag Properties:

- targetDate (Date)
- completedDate (Date) completedDate type:: Date
- progress (Number, 0-100)

# Project with milestones

- Website Redesign #Project Progress tracked via milestones:
  - Design Phase #Milestone progress:: 100 targetDate:: [2025-01-15] completedDate:: [2025-01-10]
  - Development #Milestone progress:: 60 targetDate:: [2025-02-01]
```

### Learning Workflow

```markdown
# Learning resources

- Learn TypeScript #Learning resourceType:: Course provider:: [[Udemy]] url:: https://example.com status:: In Progress
  progress:: 45

# Daily learning notes

- TypeScript Study - Day 5 #LearningLog date:: [[Today]] topic:: Interfaces resources:: [[Learn TypeScript]] notes::
  - Learned about optional properties
  - Practiced union types nextTopics::
  - Type assertions
  - Generics
```

## Command Examples

### Search Commands

```bash
# Quick add notes without losing context
Cmd+Shift+P → "Quick add"

# Jump to specific date
Cmd+Shift+P → "Go to date" → "Next Friday"

# Bulk operations
Cmd+Shift+P → "Move blocks to" → select page
Cmd+Shift+P → "Export page EDN data"
```

### Property Shortcuts

```
Cmd-p    # Add property
Cmd-j    # Quick edit properties
p a      # Toggle all properties visibility
p t      # Add/remove tags
p i      # Set icon
p s      # Set Status (tasks)
p p      # Set Priority (tasks)
p d      # Set Deadline (tasks)
```

### Navigation

```
g n      # Next journal day
g p      # Previous journal day
Cmd-k    # Search nodes
Cmd-;    # Toggle properties visibility
```

---

**Note:** This is a living document. Add your own examples as you discover new patterns!
