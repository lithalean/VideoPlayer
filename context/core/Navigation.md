# VideoPlayer Navigation Context

**Purpose**: Screen flow, user journeys, and navigation patterns  
**Version**: 1.0  
**Navigation Pattern**: Split View (iOS/macOS) + Stack (tvOS)  
**Last Updated**: August 12, 2025

## Navigation State Machine
```
┌─────────┐     Import     ┌──────────┐
│  Empty  │ ───────────────►│ Library  │
│ Library │                 │(Has Items)│
└─────────┘                 └─────┬────┘
                                  │
                    ┌─────────────┴──────────────┐
                    ▼                            ▼
            ┌──────────┐                ┌──────────┐
            │  Movies  │                │  Shows   │
            │   Grid   │                │   Grid   │
            └────┬─────┘                └────┬─────┘
                 │ Tap Poster                 │ Tap Card
                 ▼                            ▼
            ┌──────────┐                ┌──────────┐
            │MovieView │                │(ShowView)│
            │(Playback)│                │ [Future] │
            └────┬─────┘                └──────────┘
                 │ Fullscreen
                 ▼
            ┌──────────┐
            │Fullscreen│
            │  Video   │
            └──────────┘
```

## Screen Hierarchy

### iOS/macOS Structure
```
NavigationSplitView
├── Sidebar (200-300pt width)
│   ├── Library Section
│   │   ├── Movies (NavigationItem)
│   │   └── Shows (NavigationItem)
│   └── Import Controls (pinned bottom)
│       ├── Import Menu
│       │   ├── Kind Picker (Movie/Show)
│       │   └── Choose Files Button
│       └── Quick Import Button
└── Detail View
    ├── MoviesView (when Movies selected)
    ├── ShowsView (when Shows selected)
    └── Empty State (nothing selected)
```

### tvOS Structure
```
NavigationStack
└── VStack
    ├── VideoPlayerNavigationSidebar (max 360pt)
    │   ├── Library Items
    │   └── (No import - tvOS limitation)
    ├── Divider
    └── Detail View (remaining space)
        ├── MoviesView
        ├── ShowsView
        └── Empty State
```

## User Journey Flows

### First Launch → First Video
```
1. App Opens → Empty Library State
2. User sees "No Movies" message with film icon
3. Clicks Import button (iOS/macOS only)
4. Menu appears → Selects Movie or Show kind
5. "Choose .m4v Files" → System file picker
6. Selects one or multiple .m4v files
7. Import begins (async, no progress shown)
8. Poster generates at 10% timestamp
9. Video appears in appropriate grid
10. User taps poster → MovieView opens
11. Taps Play → Overlay fades, video plays
```

### Browse → Watch Flow
```
1. App opens → Sidebar shows badge counts
2. User selects Movies from sidebar
3. Grid of 2:3 posters appears
4. User scrolls to find video
5. Taps poster (tvOS: focuses then selects)
6. MovieView opens with backdrop gradient
7. Sees title and Play button overlay
8. Taps Play → UI fades out
9. Video plays fullscreen
10. Taps Back to return to grid
```

### Import Flow (iOS/macOS)
```
1. Import controls visible at sidebar bottom
2. User taps import menu
3. Selects kind: Movie or Show
4. Taps "Choose .m4v Files..."
5. System picker opens (multi-select enabled)
6. Selects files → Taps Open
7. Files validate (.m4v only)
8. Async import per file:
   - Copy to app container
   - Extract metadata
   - Generate poster
   - Save to SwiftData
9. Items appear in grid as completed
```

### Delete Flow
```
1. User in MoviesView/ShowsView
2. Long press on poster (iOS/macOS)
3. Context menu appears
4. Taps "Delete" (destructive role)
5. File removed from disk
6. Record deleted from SwiftData
7. Poster orphaned (known issue)
8. Grid updates immediately
```

## Navigation Components

### Sidebar (VideoPlayerNavigationSidebar)
```yaml
structure:
  header: "Library"
  items:
    - label: "Movies"
      icon: "film"
      badge: movieItems.count || nil
    - label: "Shows"
      icon: "tv"
      badge: showItems.count || nil
  footer: Import controls (iOS/macOS)

state:
  @State selection: NavigationItem?
  @State searchText: String (unused)
  @State showingFileImporter: Bool
  @State importKind: VideoKind

behavior:
  selection_change: Updates detail view
  badge_update: Auto via @Query
  import_trigger: Opens fileImporter sheet
```

### MoviesView
```yaml
navigation_type: Push
grid_structure:
  type: LazyVGrid
  columns: adaptive(140-220pt)
item_interaction:
  tap: NavigationLink → MovieView
  long_press: Context menu (delete)
  focus: Scale effect (tvOS)
empty_state:
  icon: "film"
  title: "No Movies"
  message: "Import .m4v files to see your movies here"
```

### MovieView (Playback Screen)
```yaml
entry: NavigationLink from grid
controls:
  back:
    position: top-left
    action: dismiss()
  play:
    position: bottom-center
    action: Start playback, fade UI
  fullscreen:
    position: top-right
    action: Platform-specific expand
exit:
  back_button: Return to grid
  gesture: None (must use button)
```

## Platform Navigation Patterns

### iOS Navigation
```swift
// Split view with sidebar
NavigationSplitView {
    Sidebar
        .navigationSplitViewColumnWidth(min: 200, ideal: 240, max: 300)
} detail: {
    DetailView
}

// Fullscreen video
.fullScreenCover(isPresented: $showFullscreen) {
    VideoPlayer(player: player)
}

// File import
.fileImporter(
    isPresented: $showingFileImporter,
    allowedContentTypes: [.mpeg4Movie],
    allowsMultipleSelection: true
)
```

### tvOS Navigation
```swift
// Single stack (no split)
NavigationStack {
    VStack {
        Sidebar.frame(maxHeight: 360)
        Divider()
        DetailView
    }
}

// Focus-based selection
@FocusState private var isFocused: Bool
.focusable(true)
.focused($isFocused)

// No file import available
// No context menus
// Button-based controls only
```

### macOS Navigation
```swift
// Split view with sidebar
NavigationSplitView {
    Sidebar // With traffic light spacing
} detail: {
    DetailView
}

// Window fullscreen
NSApp.windows.first?.toggleFullScreen(nil)

// Native context menus
.contextMenu { ... }

// Hover states enabled
```

## Navigation State Management

### Selection State
```swift
// Shared between sidebar and detail
@State private var selection: NavigationItem? = 
    NavigationItem(label: "Movies", systemImage: "film")

// NavigationItem is Hashable for selection
```

### Query-Driven Updates
```swift
// Badges update automatically
@Query(filter: #Predicate<VPVideo> { $0.kindRaw == "movie" })
private var movieItems: [VPVideo]

// Used in navigation items
badge: movieItems.isEmpty ? nil : movieItems.count
```

### Import State
```swift
@State private var showingFileImporter = false
@State private var importKind: VideoKind = .movie
@Environment(\.modelContext) private var modelContext
```

## Empty States

### No Movies
- **Icon**: SF Symbol "film"
- **Title**: "No Movies"
- **Message**: "Import .m4v files to see your movies here"
- **Action**: None (passive)

### No Shows
- **Icon**: SF Symbol "tv"
- **Title**: "No Shows"  
- **Message**: "Import .m4v files to see your shows here"
- **Action**: None (passive)

### Nothing Selected
- **Icon**: None
- **Title**: "Select an item"
- **Message**: None
- **Style**: .secondary foreground

## Navigation Transitions

| From | To | Trigger | Animation | Type |
|------|-----|---------|-----------|------|
| Sidebar | MoviesView | Select "Movies" | None | Replace detail |
| Sidebar | ShowsView | Select "Shows" | None | Replace detail |
| MoviesView | MovieView | Tap poster | Default push | NavigationLink |
| ShowsView | (Future) | Tap show card | Default push | NavigationLink |
| MovieView | Fullscreen | Tap button | Sheet/Window | Modal |
| Any | File Picker | Import button | Sheet slide | Modal |
| Grid | Context Menu | Long press | Popup | Popover |
| MovieView | Grid | Back button | Default pop | Navigation pop |

## Focus Navigation (tvOS)

### Focus Flow
```
Sidebar Items → Grid Items → Playback Controls
     ↑              ↓              ↓
     └──────────────←──────────────┘
```

### Focus Guides
- Sidebar: Vertical navigation
- Grid: 2D navigation with guide rails
- Playback: Horizontal button focus

### Remote Buttons
- **Menu**: Back/dismiss
- **Play/Pause**: Toggle playback
- **Select**: Activate focused item
- **Directional**: Move focus

## Keyboard Navigation (macOS)

### Shortcuts (Future)
- **Space**: Play/Pause
- **←/→**: Seek -10/+10 seconds
- **↑/↓**: Volume up/down
- **F**: Toggle fullscreen
- **Esc**: Exit fullscreen
- **⌘O**: Import files
- **Delete**: Delete selected

## Navigation Performance

### Optimized
- Lazy loading grids
- Single detail view instance
- Shallow navigation stack
- No unnecessary re-renders

### Needs Work
- No deep linking
- No state restoration
- No navigation history
- No breadcrumbs

## Future Navigation Enhancements

1. **ShowView**: Individual show playback screen
2. **Search Results**: Filtered grid view
3. **Settings**: Preferences screen
4. **Queue**: Multi-video playlist
5. **History**: Recently watched
6. **Filters**: Genre/year/duration
7. **Sort Options**: Name/date/duration
8. **Deep Linking**: URL scheme support
9. **State Restoration**: Resume where left off
10. **Keyboard Navigation**: Full keyboard control