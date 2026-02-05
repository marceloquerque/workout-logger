---
name: Workout Logger iOS App
overview: Build a minimal SwiftUI workout logging app with SwiftData persistence, calendar view with workout dots, and markdown notes editor. CloudKit sync deferred for later.
todos:
  - id: p1-xcode-project
    content: "[Phase 1] Create Xcode project 'WorkoutLogger' with iOS/SwiftUI template and folder structure"
    status: pending
  - id: p1-workout-model
    content: "[Phase 1] Create Workout.swift SwiftData model (id, type, date, createdAt, editedAt, notes)"
    status: pending
  - id: p1-workout-type
    content: "[Phase 1] Create WorkoutType.swift enum (Upper/Lower/Full Body with colors)"
    status: pending
  - id: p1-date-extensions
    content: "[Phase 1] Create DateExtensions.swift (timeOnly, dateOnly, fullTimestamp formatters)"
    status: pending
  - id: p1-app-entry
    content: "[Phase 1] Set up WorkoutLoggerApp.swift with SwiftData container"
    status: pending
  - id: p2-calendar-grid
    content: "[Phase 2] Build CalendarGridView.swift - month grid with date cells"
    status: pending
  - id: p2-swipe-nav
    content: "[Phase 2] Add horizontal swipe gesture navigation to CalendarGridView"
    status: pending
  - id: p2-calendar-view
    content: "[Phase 2] Build CalendarView.swift - main screen shell combining calendar + list placeholder"
    status: pending
  - id: p2-date-selection
    content: "[Phase 2] Implement selectedDate state and pass to child views"
    status: pending
  - id: p2-workout-dots
    content: "[Phase 2] Add colored workout dots under dates (query month, group by date, max 3 dots)"
    status: pending
  - id: p3-workout-row
    content: "[Phase 3] Build WorkoutRowView.swift - type label, time, tap gesture"
    status: pending
  - id: p3-workout-list
    content: "[Phase 3] Build WorkoutListView.swift - filtered list + 'No Workouts' empty state"
    status: pending
  - id: p3-type-picker
    content: "[Phase 3] Build WorkoutTypePickerView.swift - modal sheet with 3 type buttons"
    status: pending
  - id: p3-editor
    content: "[Phase 3] Build WorkoutEditorView.swift - TextEditor with autosave on change"
    status: pending
  - id: p3-navigation
    content: "[Phase 3] Wire up navigation: FAB -> picker -> editor, row tap -> editor"
    status: pending
  - id: p4-clipboard
    content: "[Phase 4] Create ClipboardManager.swift with Type||||Notes format"
    status: pending
  - id: p4-copy-menu
    content: "[Phase 4] Add 'Copy Workout' context menu to WorkoutRowView"
    status: pending
  - id: p4-paste-button
    content: "[Phase 4] Add paste button to WorkoutTypePickerView when clipboard has workout"
    status: pending
  - id: p4-pull-reveal
    content: "[Phase 4] Add pull-to-reveal date header in WorkoutEditorView"
    status: pending
  - id: p4-type-dropdown
    content: "[Phase 4] Add workout type dropdown in editor navigation bar"
    status: pending
  - id: p4-delete-menu
    content: "[Phase 4] Add '...' menu with Delete action in WorkoutEditorView"
    status: pending
  - id: p4-swipe-dismiss
    content: "[Phase 4] Implement swipe-right-to-dismiss gesture in WorkoutEditorView"
    status: pending
  - id: p5-markdown
    content: "[Phase 5] Add markdown rendering with native AttributedString"
    status: pending
  - id: p5-haptics
    content: "[Phase 5] Add haptic feedback (date selection, type selection, copy action)"
    status: pending
  - id: p5-accessibility
    content: "[Phase 5] Add accessibility labels to all interactive elements"
    status: pending
  - id: p5-dark-mode
    content: "[Phase 5] Test and verify dark mode appearance"
    status: pending
  - id: p5-edge-cases
    content: "[Phase 5] Test edge cases (empty states, invalid clipboard, timezone handling)"
    status: pending
isProject: false
---

# Workout Logger iOS App Implementation

## Summary

Build a minimal iOS workout logger using SwiftUI + SwiftData. Calendar-based UI showing workout dots, free-form markdown notes, copy/paste between dates. CloudKit deferred.

## Tech Stack

- Swift 6.2+, iOS 26.1+
- SwiftUI for UI
- SwiftData for local persistence (CloudKit-ready for later)
- Native `AttributedString` for markdown rendering

## Architecture

```mermaid
flowchart TB
    subgraph Views
        CalendarView --> CalendarGridView
        CalendarView --> WorkoutListView
        WorkoutListView --> WorkoutRowView
        CalendarView --> WorkoutTypePickerView
        WorkoutTypePickerView --> WorkoutEditorView
        WorkoutRowView --> WorkoutEditorView
    end

    subgraph Data
        WorkoutModel[Workout Model]
        WorkoutType[WorkoutType Enum]
    end

    subgraph Utilities
        DateExtensions
        ClipboardManager
    end

    Views --> Data
    Views --> Utilities
```

## File Structure

```
WorkoutLogger/
├── WorkoutLoggerApp.swift
├── Models/
│   └── Workout.swift
├── Views/
│   ├── CalendarView.swift
│   ├── WorkoutListView.swift
│   ├── WorkoutTypePickerView.swift
│   ├── WorkoutEditorView.swift
│   └── Components/
│       ├── CalendarGridView.swift
│       └── WorkoutRowView.swift
├── Utilities/
│   ├── WorkoutType.swift
│   ├── DateExtensions.swift
│   └── ClipboardManager.swift
└── Resources/
    └── Assets.xcassets
```

## Execution Order & Parallelization

```mermaid
flowchart LR
    subgraph Phase1 [Phase 1: Foundation]
        P1[p1-xcode-project] --> P1a[p1-workout-model]
        P1a --> P1b[p1-workout-type]
        P1b --> P1c[p1-date-extensions]
        P1c --> P1d[p1-app-entry]
    end

    subgraph Phase2 [Phase 2: Calendar]
        P2a[p2-calendar-grid]
        P2a --> P2b[p2-swipe-nav]
        P2a --> P2c[p2-calendar-view]
        P2c --> P2d[p2-date-selection]
        P2d --> P2e[p2-workout-dots]
    end

    subgraph Phase3 [Phase 3: CRUD]
        P3a[p3-workout-row]
        P3b[p3-workout-list]
        P3c[p3-type-picker]
        P3d[p3-editor]
        P3a --> P3e[p3-navigation]
        P3b --> P3e
        P3c --> P3e
        P3d --> P3e
    end

    subgraph Phase4 [Phase 4: Features]
        P4a[p4-clipboard]
        P4a --> P4b[p4-copy-menu]
        P4a --> P4c[p4-paste-button]
        P4d[p4-pull-reveal]
        P4e[p4-type-dropdown]
        P4f[p4-delete-menu]
        P4g[p4-swipe-dismiss]
    end

    subgraph Phase5 [Phase 5: Polish]
        P5a[p5-markdown]
        P5b[p5-haptics]
        P5c[p5-accessibility]
        P5d[p5-dark-mode]
        P5e[p5-edge-cases]
    end

    Phase1 --> Phase2
    Phase1 --> Phase3
    Phase2 --> Phase4
    Phase3 --> Phase4
    Phase4 --> Phase5
```

### Parallelization Summary

| Phase   | Execution       | Notes                                                                          |
| ------- | --------------- | ------------------------------------------------------------------------------ |
| Phase 1 | Sequential      | Must complete first; creates project foundation                                |
| Phase 2 | After Phase 1   | Can run in parallel with Phase 3                                               |
| Phase 3 | After Phase 1   | Can run in parallel with Phase 2; p3-a/b/c/d are parallel, p3-e depends on all |
| Phase 4 | After Phase 2+3 | p4-a blocks p4-b/c; p4-d/e/f/g are independent                                 |
| Phase 5 | After Phase 4   | All tasks can run in parallel                                                  |

---

## Implementation Phases (Detailed)

### Phase 1: Foundation (Sequential)

| Task ID            | Description                                | Depends On         |
| ------------------ | ------------------------------------------ | ------------------ |
| p1-xcode-project   | Create Xcode project with folder structure | -                  |
| p1-workout-model   | Create Workout.swift SwiftData model       | p1-xcode-project   |
| p1-workout-type    | Create WorkoutType.swift enum              | p1-workout-model   |
| p1-date-extensions | Create DateExtensions.swift                | p1-workout-type    |
| p1-app-entry       | Set up WorkoutLoggerApp.swift              | p1-date-extensions |

### Phase 2: Calendar UI

| Task ID           | Description                       | Depends On        |
| ----------------- | --------------------------------- | ----------------- |
| p2-calendar-grid  | Build CalendarGridView month grid | Phase 1           |
| p2-swipe-nav      | Add swipe gesture navigation      | p2-calendar-grid  |
| p2-calendar-view  | Build CalendarView main screen    | p2-calendar-grid  |
| p2-date-selection | Implement selectedDate state      | p2-calendar-view  |
| p2-workout-dots   | Add colored dots under dates      | p2-date-selection |

### Phase 3: Workout List & CRUD

| Task ID         | Description                           | Depends On                                                 |
| --------------- | ------------------------------------- | ---------------------------------------------------------- |
| p3-workout-row  | Build WorkoutRowView component        | Phase 1                                                    |
| p3-workout-list | Build WorkoutListView + empty state   | Phase 1                                                    |
| p3-type-picker  | Build WorkoutTypePickerView modal     | Phase 1                                                    |
| p3-editor       | Build WorkoutEditorView with autosave | Phase 1                                                    |
| p3-navigation   | Wire up all navigation flows          | p3-workout-row, p3-workout-list, p3-type-picker, p3-editor |

### Phase 4: Advanced Features

| Task ID          | Description                 | Depends On   |
| ---------------- | --------------------------- | ------------ |
| p4-clipboard     | Create ClipboardManager     | Phase 3      |
| p4-copy-menu     | Add copy context menu       | p4-clipboard |
| p4-paste-button  | Add paste button to picker  | p4-clipboard |
| p4-pull-reveal   | Pull-to-reveal date header  | Phase 3      |
| p4-type-dropdown | Type dropdown in editor nav | Phase 3      |
| p4-delete-menu   | Delete action in ... menu   | Phase 3      |
| p4-swipe-dismiss | Swipe-right-to-dismiss      | Phase 3      |

### Phase 5: Polish (All Parallel)

| Task ID          | Description          | Depends On |
| ---------------- | -------------------- | ---------- |
| p5-markdown      | Markdown rendering   | Phase 4    |
| p5-haptics       | Haptic feedback      | Phase 4    |
| p5-accessibility | Accessibility labels | Phase 4    |
| p5-dark-mode     | Dark mode testing    | Phase 4    |
| p5-edge-cases    | Edge case testing    | Phase 4    |

---

## Key Implementation Details

**SwiftData Model** (`Workout.swift`):

- `date` normalized to start of day for grouping
- `createdAt`/`editedAt` preserve actual timestamps

**Calendar Dots Query**:

- Query workouts for visible month range only
- Group by date, show max 3 dots per date

**Autosave**:

- `onChange(of: notes)` triggers save
- Optional 300ms debounce for performance

**Copy/Paste Format**:

- Clipboard string: `"Upper||||Notes content here"`
- Parse on picker appear, show paste button if valid

---

## Notes

- CloudKit can be enabled later by adding `.modelContainer(for: Workout.self, isCloudKitEnabled: true)` and enabling iCloud capability
- No third-party dependencies needed for v1
- When delegating to agents, provide the spec file and relevant task context
