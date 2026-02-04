# Workout Logger - iOS App Specification

## Overview

Build a minimal, focused iOS workout logging app using SwiftUI and SwiftData with iCloud sync. The app replaces using iOS Calendar for workout logging by providing a clean, distraction-free interface centered around free-form note-taking with light structure.

## Core Philosophy

- Dead simple, minimal design
- No unnecessary features or analytics
- Focus on the notes content (like iOS Notes app)
- Apple HIG-obsessed design patterns
- Zero friction on first interaction

---

## Tech Stack

- **Language**: Swift 5.0+ (targeting Swift 6.2+)
- **Platform**: iOS 26.1+ (iPhone)
- **UI Framework:** SwiftUI
- **Data Persistence:** SwiftData
- **Cloud Sync:** CloudKit (integrated via SwiftData)
- **Markdown:** Native SwiftUI AttributedString or lightweight package (e.g., MarkdownUI)

---

## Project Structure

```
WorkoutLogger/
├── WorkoutLoggerApp.swift              // App entry point
├── Models/
│   └── Workout.swift                   // SwiftData model
├── Views/
│   ├── CalendarView.swift              // Main screen
│   ├── WorkoutListView.swift           // List of workouts for selected day
│   ├── WorkoutTypePickerView.swift     // Type selection modal
│   ├── WorkoutEditorView.swift         // Notes editor
│   └── Components/
│       ├── CalendarGridView.swift      // Monthly calendar with dots
│       └── WorkoutRowView.swift        // Single workout row
├── ViewModels/
│   └── CalendarViewModel.swift         // Selected date & filtering
├── Utilities/
│   ├── WorkoutType.swift               // Enum for types & colors
│   ├── DateExtensions.swift            // Date helpers
│   └── ClipboardManager.swift          // Copy/paste logic
└── Resources/
    └── Assets.xcassets
```

---

## Data Model

### Workout (SwiftData Model)

```swift
import SwiftData
import Foundation

@Model
final class Workout {
    var id: UUID
    var type: String              // "Upper", "Lower", "Full Body"
    var date: Date                // Calendar day (normalized to start of day)
    var createdAt: Date           // Creation timestamp (with time)
    var editedAt: Date            // Last edit timestamp (with time)
    var notes: String             // Markdown-formatted notes

    init(type: String, date: Date, notes: String = "") {
        self.id = UUID()
        self.type = type
        self.date = Calendar.current.startOfDay(for: date)
        self.createdAt = Date()
        self.editedAt = Date()
        self.notes = notes
    }
}
```

**Key Points:**

- `date` is normalized to start of day (12:00 AM) for calendar grouping
- `createdAt` and `editedAt` preserve actual timestamps
- A workout is permanently tied to its creation date (no moving, only copy/paste)

---

## Workout Types

```swift
enum WorkoutType: String, CaseIterable {
    case upper = "Upper"
    case lower = "Lower"
    case fullBody = "Full Body"

    var color: Color {
        switch self {
        case .upper: return .blue
        case .lower: return .orange
        case .fullBody: return .green
        }
    }
}
```

---

## App Entry Point

### WorkoutLoggerApp.swift

```swift
import SwiftUI
import SwiftData

@main
struct WorkoutLoggerApp: App {
    var body: some Scene {
        WindowGroup {
            CalendarView()
        }
        .modelContainer(for: Workout.self, isCloudKitEnabled: true)
    }
}
```

**CloudKit Setup:**

- Enable CloudKit in Xcode capabilities
- SwiftData handles sync automatically with `isCloudKitEnabled: true`
- Conflicts resolved via last-write-wins (default)

---

## Screen Specifications

### 1. CalendarView (Main Screen)

**Layout:**

```
┌─────────────────────────────────────┐
│  < February 2026           >        │  ← Month navigation
│                                     │
│  Su Mo Tu We Th Fr Sa               │
│         1  2  3  4  5  6            │
│   7  8  9 •● 11 12 13               │  ← Colored dots under dates
│  14 15 16 17 18 19 20               │
│  21 22 23 24 25 26 27               │
│  28                                 │
│                                     │
├─────────────────────────────────────┤
│  Wednesday, Feb 4                   │  ← Selected date header
│                                     │
│  Upper              2:34 PM  ───┐   │  ← Workout rows
│                                  │   │
│  Lower              5:15 PM  ───┤   │
│                                  │   │
│  [Empty state or more rows]  ───┘   │
│                                     │
│                              [+]    │  ← Floating action button
└─────────────────────────────────────┘
```

**Components:**

**CalendarGridView:**

- Display current month grid
- Tap date → update `selectedDate` state
- Show colored dots under dates that have workouts:
  - Query all workouts for visible month
  - Group by date
  - Stack dots horizontally (max 3 visible, if more show ellipsis)
  - Each dot colored by workout type
- Left/right arrows to navigate months (or swipe gestures)

**WorkoutListView:**

- Display workouts for `selectedDate`
- Query: `@Query(filter: #Predicate<Workout> { $0.date == selectedDate }, sort: \Workout.createdAt)`
- Each row is a `WorkoutRowView`
- Long-press gesture on row → show context menu with "Copy Workout"
- Empty state when no workouts (just blank space)

**Floating "+" Button:**

- Positioned bottom-right corner
- Tap → present `WorkoutTypePickerView` as modal sheet

**Implementation Notes:**

- Use `@State private var selectedDate: Date = Date()` in CalendarView
- Pass `selectedDate` to WorkoutListView
- Use `@Environment(\.modelContext)` for SwiftData operations

---

### 2. WorkoutTypePickerView (Modal Sheet)

**Layout:**

```
┌─────────────────────────────────────┐
│  ┌─────────────────────────────┐   │
│  │                             │   │
│  │         Upper               │   │  ← Large tappable button (blue)
│  │                             │   │
│  └─────────────────────────────┘   │
│                                     │
│  ┌─────────────────────────────┐   │
│  │                             │   │
│  │         Lower               │   │  ← (orange)
│  │                             │   │
│  └─────────────────────────────┘   │
│                                     │
│  ┌─────────────────────────────┐   │
│  │                             │   │
│  │       Full Body             │   │  ← (green)
│  │                             │   │
│  └─────────────────────────────┘   │
│                                     │
│  ┌─────────────────────────────┐   │  ← Only if clipboard has workout
│  │                             │   │
│  │   Paste from "Upper"        │   │  ← (gray, with paste icon)
│  │                             │   │
│  └─────────────────────────────┘   │
└─────────────────────────────────────┘
```

**Behavior:**

- Present as `.sheet` modifier
- Check clipboard on appear:
  - If clipboard contains workout data (format: `"Type||||Notes"`), show 4th "Paste" button
  - Extract type from clipboard for display
- On type selection:
  - Create new `Workout` instance with selected type and `selectedDate`
  - If pasting, also copy notes from clipboard
  - Insert into SwiftData context
  - Dismiss sheet
  - Present `WorkoutEditorView` with new workout

**Implementation:**

```swift
.sheet(isPresented: $showingTypePicker) {
    WorkoutTypePickerView(
        selectedDate: selectedDate,
        onSelect: { workout in
            // Navigate to editor with this workout
        }
    )
}
```

---

### 3. WorkoutEditorView (Full-screen)

**Layout:**

```
┌─────────────────────────────────────┐
│  <  Upper ▾                    ...  │  ← Navigation bar
├─────────────────────────────────────┤
│                                     │
│  [Notes field - full screen]        │
│                                     │
│  User types here with Markdown      │
│  support. Autosaves on every        │
│  keystroke.                         │
│                                     │
│                                     │
│                                     │
│  Pull down to reveal:               │
│  ┌─────────────────────────────┐   │
│  │ Created Feb 4, 2:34 PM      │   │  ← Pull-to-reveal header
│  └─────────────────────────────┘   │
│                                     │
└─────────────────────────────────────┘
```

**Navigation Bar:**

- **Leading (Left):**

  - `<` Back button
  - Swipe right gesture also triggers back
  - Both auto-save and dismiss

- **Center:**

  - `"Upper ▾"` (workout type with dropdown indicator)
  - Tappable → present dropdown menu with workout type options
  - On selection → update workout type, save, update UI

- **Trailing (Right):**
  - `...` Ellipsis menu button
  - Future actions: Delete, Duplicate, etc. (implement basic menu for now)

**Pull-to-Reveal Date Header:**

- When user pulls down on notes field (scrolls beyond top), reveal date header
- Shows: `"Created Feb 4, 2:34 PM"` (gray text, small font)
- Tappable → cycles between:
  - `"Created [date]"`
  - `"Edited [date]"` (if different from created)
- Use `ScrollView` with `safeAreaInset(edge: .top)` for pull-to-reveal effect

**Notes Field:**

- `TextEditor` binding to `workout.notes`
- Full screen height
- User must tap to focus (keyboard not auto-shown)
- Markdown formatting:
  - In editor: plain text input
  - When reopened: render markdown using `Text(AttributedString(markdown: notes))`
  - Or use MarkdownUI package for live rendering while typing

**Autosave:**

```swift
.onChange(of: workout.notes) { oldValue, newValue in
    workout.editedAt = Date()
    // Debounce optional (0.3s delay) to avoid excessive saves
    try? modelContext.save()
}
```

**Swipe Right to Dismiss:**

```swift
.gesture(
    DragGesture()
        .onEnded { gesture in
            if gesture.translation.width > 100 {
                // Save and dismiss
                dismiss()
            }
        }
)
```

**Workout Type Dropdown:**

- Use `.confirmationDialog` or custom dropdown
- Show all workout types
- On selection → update `workout.type`, save

---

### 4. WorkoutRowView (Component)

**Layout:**

```
Upper                                    2:34 PM
```

- Left: Workout type (with type color as accent)
- Right: Creation time (formatted as "h:mm a")
- Long-press → show context menu with "Copy Workout"

**Styling:**

- Use SF Symbols for subtle type indicators (optional)
- Divider between rows
- Tap row → navigate to `WorkoutEditorView` with that workout

---

## Key Features Implementation

### Copy/Paste Workflow

**Clipboard Format:**

```
"WorkoutType||||Notes content here"
```

**Copy (in WorkoutRowView):**

```swift
.contextMenu {
    Button("Copy Workout") {
        let clipboardData = "\(workout.type)||||\(workout.notes)"
        UIPasteboard.general.string = clipboardData
    }
}
```

**Paste Detection (in WorkoutTypePickerView):**

```swift
@State private var clipboardWorkout: (type: String, notes: String)?

var body: some View {
    VStack {
        // Type buttons...

        if let clipboard = clipboardWorkout {
            Button("Paste from \"\(clipboard.type)\"") {
                // Create workout with clipboard data
            }
        }
    }
    .onAppear {
        if let data = UIPasteboard.general.string,
           data.contains("||||") {
            let parts = data.split(separator: "||||", maxSplits: 1)
            if parts.count == 2 {
                clipboardWorkout = (
                    type: String(parts[0]),
                    notes: String(parts[1])
                )
            }
        }
    }
}
```

---

### Calendar Dots Implementation

**Query workouts for visible month:**

```swift
@Query(filter: #Predicate<Workout> { workout in
    workout.date >= monthStart && workout.date <= monthEnd
}) var monthWorkouts: [Workout]
```

**Group by date and display dots:**

```swift
// In CalendarGridView, for each date cell:
let workoutsForDate = monthWorkouts.filter {
    Calendar.current.isDate($0.date, inSameDayAs: date)
}

HStack(spacing: 2) {
    ForEach(workoutsForDate.prefix(3)) { workout in
        Circle()
            .fill(WorkoutType(rawValue: workout.type)?.color ?? .gray)
            .frame(width: 4, height: 4)
    }
    if workoutsForDate.count > 3 {
        Text("...")
            .font(.caption2)
    }
}
```

---

### Date Formatting Helpers

```swift
extension Date {
    func formatted(as format: String) -> String {
        let formatter = DateFormatter()
        formatter.dateFormat = format
        return formatter.string(from: self)
    }

    var timeOnly: String {
        formatted(as: "h:mm a")
    }

    var dateOnly: String {
        formatted(as: "MMM d, yyyy")
    }

    var fullTimestamp: String {
        formatted(as: "MMM d, h:mm a")
    }
}
```

---

## UI/UX Requirements

### Design Principles

- Minimal chrome, maximum content focus
- Native iOS patterns (pull-to-refresh, swipe gestures, etc.)
- No unnecessary animations (keep it snappy)
- Respect user's Dynamic Type settings
- Dark mode support (automatic via SwiftUI)

### Accessibility

- All interactive elements must have accessibility labels
- Support VoiceOver navigation
- Ensure sufficient color contrast for workout type colors
- Support Dynamic Type for all text

### Haptics

- Light haptic feedback on:
  - Date selection in calendar
  - Workout type selection
  - Copy action completion

---

## Implementation Phases

### Phase 1: Foundation (MVP)

1. Set up Xcode project with SwiftData and CloudKit
2. Create `Workout` model
3. Build `CalendarGridView` (static month display, no data)
4. Build `WorkoutListView` with mock data
5. Implement basic navigation between views

### Phase 2: Core CRUD

6. Implement workout creation flow (picker → editor)
7. Implement autosave in editor
8. Wire up calendar date selection to filter workout list
9. Implement tap on workout row → open editor

### Phase 3: Advanced Features

10. Add calendar dots based on workout types
11. Implement copy/paste functionality
12. Add pull-to-reveal date in editor
13. Implement swipe-right-to-dismiss

### Phase 4: Polish

14. Add Markdown rendering in editor
15. Implement workout type dropdown in editor nav bar
16. Implement "..." menu (delete workout minimum)
17. Test CloudKit sync across devices
18. Add haptics and animations
19. Accessibility audit
20. Dark mode testing

---

## Testing Strategy

### Unit Tests

- Date normalization logic (start of day)
- Workout filtering by date
- Clipboard data parsing

### UI Tests

- Create workout flow (picker → editor → save)
- Copy/paste workflow
- Calendar date selection updates list
- Autosave behavior

### Manual Testing

- CloudKit sync (2 devices or simulator + physical device)
- Offline behavior (create workouts without internet, sync when online)
- Conflict resolution (edit same workout on 2 devices)

---

## Known Limitations & Future Considerations

**Current Scope (v1.0):**

- iOS only (no macOS, watchOS, iPad optimizations)
- No search functionality
- No workout analytics or trends
- No export/import
- No workout templates
- No customizable workout types (only Upper/Lower/Full Body)

**Potential Future Features:**

- Search workouts by content or type
- Export to PDF or Markdown file
- Custom workout types
- Workout templates for quick logging
- iPad-optimized layout (master-detail)
- Widgets for quick logging

---

## Critical Implementation Notes

### SwiftData Best Practices

- Always use `@MainActor` for UI-related model updates
- Use `modelContext.save()` explicitly after batch operations
- Handle errors gracefully (e.g., CloudKit sync failures)

### CloudKit Considerations

- Test with both WiFi and cellular data
- Handle merge conflicts (default: last-write-wins)
- User must be signed into iCloud
- Show appropriate error messages if iCloud unavailable

### Performance

- Calendar dots: optimize query to only fetch month range
- Autosave: consider debouncing (300ms delay) to avoid excessive writes
- List rendering: use lazy loading for large workout lists

### Edge Cases

- Empty states (no workouts for selected day)
- First launch (empty calendar, prompt to create first workout)
- Deleting a workout currently being edited
- Clipboard contains invalid data format
- Date/time edge cases (timezone changes, DST)

---

## Deliverables

**Xcode Project Structure:**

- Organized file structure as specified above
- SwiftData model with CloudKit enabled
- All views implemented with specified functionality
- Basic unit tests for core logic
- README with setup instructions

**App Functionality:**

- Create, edit, delete workouts
- Calendar view with workout dots
- Copy/paste workouts between dates
- Autosave with offline support
- CloudKit sync across devices
- Markdown support in notes

**Code Quality:**

- Clean, readable code with comments
- Follow Swift naming conventions
- No force unwraps (use proper optional handling)
- Accessibility labels on all interactive elements
- Dark mode support

---

## Getting Started for AI Agent

**Steps to Build:**

1. Create new Xcode project: "WorkoutLogger" (iOS, SwiftUI, Swift)

2. Enable capabilities:

   - iCloud → CloudKit
   - Background Modes → Remote notifications (for CloudKit)

3. Create file structure as specified above

4. Implement in this order:

   - Data model (`Workout.swift`, `WorkoutType.swift`)
   - App entry point with SwiftData container
   - CalendarView shell (no data)
   - WorkoutListView with mock data
   - WorkoutTypePickerView
   - WorkoutEditorView with autosave
   - Wire up navigation and data flow
   - Add calendar dots
   - Implement copy/paste
   - Polish UI and add remaining features

5. Test thoroughly:
   - Create multiple workouts
   - Copy/paste between dates
   - Test on 2 devices for CloudKit sync
   - Dark mode
   - Accessibility with VoiceOver

---

## Questions for Clarification

If anything is unclear during implementation:

1. **Markdown rendering**: Prefer native SwiftUI or MarkdownUI package?
2. **Calendar navigation**: Swipe gestures or arrow buttons only?
3. **Empty state design**: Show placeholder text or just blank space?
4. **Error handling**: Silent failures or user-facing alerts for CloudKit errors?
5. **"..." menu actions**: Just delete for now, or implement duplicate as well?

Default to Apple HIG patterns and iOS Notes app behavior when in doubt.

---

## Success Criteria

The app is complete when:

✅ User can create workouts with type and notes
✅ Calendar shows colored dots for workout days
✅ Tapping a date filters the workout list
✅ Notes autosave on every keystroke
✅ Swipe right to exit editor
✅ Copy/paste workflow works between dates
✅ Markdown renders correctly in editor
✅ CloudKit syncs workouts across devices
✅ No crashes or data loss
✅ App feels fast and responsive (like Notes app)

---

**End of Specification**

This document should provide all necessary information to build the Workout Logger app. Follow the implementation phases, adhere to the design specifications, and prioritize the core CRUD functionality before adding polish features.
