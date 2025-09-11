# VideoPlayer Navigation Context

**Purpose**: Document the Inspector-based navigation system and user flows  
**Version**: 2.0  
**Navigation Pattern**: Inspector Panel + Detail View  
**Last Updated**: September 2025

## Navigation Architecture

### Core Structure
```
┌─────────────────────────────────────────────────┐
│                  ContentView                     │
│  ┌──────────────┐ ┌──────────────────────────┐  │
│  │ Inspector    │ │      Detail View         │  │
│  │   Panel      │ │                          │  │
│  │              │ │  ┌──────────────────┐    │  │
│  │ ┌──────────┐ │ │  │                  │    │  │
│  │ │Outliner  │ │ │  │   MoviesView     │    │  │
│  │ │   75%    │ │ │  │       or         │    │  │
│  │ │          │ │ │  │   ShowsView      │    │  │
│  │ └──────────┘ │ │  │       or         │    │  │
│  │              │ │  │   MovieView      │    │  │
│  │ ┌──────────┐ │ │  │   (playback)     │    │  │
│  │ │Import    │ │ │  │                  │    │  │
│  │ │   25%    │ │ │  └──────────────────┘    │  │
│  │ └──────────┘ │ │                          │  │
│  └──────────────┘ └──────────────────────────┘  │
│     280-320pt            Remaining space         │
└─────────────────────────────────────────────────┘
```

## Platform-Specific Navigation

### iOS/macOS NavigationSplitView
```swift
NavigationSplitView(columnVisibility: .constant(.automatic)) {
    InspectorPanel()
        .navigationSplitViewColumnWidth(min: 200, ideal: 240, max: 300)
} detail: {
    detailView  // MoviesView, ShowsView, or placeholder
}
```

### tvOS NavigationStack
```swift
NavigationStack {
    VStack(spacing: 0) {
        VideoPlayerNavigationSidebar(selection: $selection)
            .frame(maxHeight: 360)
        Divider()
        detailView
    }
}
```

## User Journey Flows

### First Launch Flow
```
1. App Opens
   └── Empty Documents/VideoPlayer/
       ├── Auto-creates "Movies" folder
       ├── Auto-creates "TV Shows" folder
       └── Shows empty state in Detail View

2. User Interaction Options:
   a. iOS: Tap Import → Document Picker → Select folder
   b. macOS: Browse filesystem directly
   c. tvOS: Limited to sandbox (no import)
```

### Browse → Play Flow
```
1. InspectorPanel (Left)
   └── OutlinerView
       └── FileBrowser
           ├── Documents/
           └── VideoPlayer/
               ├── 📁 Movies/      → Tap → Navigate to MoviesView
               │   └── movie.m4v  → Tap → Play directly
               └── 📁 TV Shows/   → Tap → Navigate to ShowsView
                   └── show.m4v   → Tap → Play directly

2. Detail View Updates
   - Folder selected → Show grid view
   - File selected → Open MovieView with AVPlayer
```

### iOS Security-Scoped Flow
```
1. ImportView (bottom 25% of Inspector)
   └── "Import" button (misnamed - actually bookmarks)
       └── Document Picker opens
           └── Select external folder
               ├── Security-scoped URL obtained
               ├── Bookmark created and saved
               └── Folder appears in FileBrowser
```

## Component Navigation Details

### InspectorPanel
**Width**: 320pt (macOS) / 280pt (iOS)  
**Split**: 75% OutlinerView / 25% ImportView  
**Style**: Glass material with floating panel appearance

```swift
struct InspectorPanel: View {
    var body: some View {
        VStack(spacing: 0) {
            // Top 75%: File browser
            OutlinerView()
                .frame(height: geometry.size.height * 0.75)
            
            // Bottom 25%: Import/bookmark controls
            ImportView(onImport: { url in
                // Create bookmark for external folder
            })
                .frame(height: geometry.size.height * 0.25)
        }
    }
}
```

### FileBrowser Navigation

**Navigation Destinations**:
```swift
enum Destination {
    case movies    // → MoviesView
    case shows     // → ShowsView  
    case file(URL) // → Direct playback
}
```

**Folder Recognition**:
```swift
if item.isDirectory {
    let lowerName = item.displayName.lowercased()
    if lowerName == "movies" {
        onNavigate?(.movies)  // → MoviesView in Detail
    } else if lowerName == "tv shows" {
        onNavigate?(.shows)   // → ShowsView in Detail
    } else {
        // Expand folder in place
        isExpanded.toggle()
    }
} else {
    // File selected - play directly
    currentMedia = item.url
    onNavigate?(.file(item.url))
}
```

### Detail View Routing

```swift
@ViewBuilder
private var detailView: some View {
    if let selection {
        switch selection.label {
        case "Movies":
            MoviesView()  // Grid of movie posters
        case "Shows":
            ShowsView()   // Grid of show cards
        default:
            Text("Select an item")
                .foregroundStyle(.secondary)
        }
    }
}
```

## Navigation State Management

### Current Implementation (Hybrid)
```swift
// Legacy: NavigationItem selection
@State private var selection: NavigationItem? = 
    NavigationItem(label: "Movies", systemImage: "film")

// New: Direct URL navigation
@State private var currentMedia: URL? = nil

// File browser state
@StateObject private var fileManager = MediaFileManager()
@Published var rootItems: [MediaFileItem] = []
@Published var expandedFolders: Set<URL> = []
```

### State Synchronization Issues
- FileBrowser and Detail View not fully synchronized
- Legacy NavigationItem still used for main routing
- Direct file selection bypasses navigation state

## User Interface Elements

### Inspector Components

**OutlinerView** (75% of Inspector):
- Contains FileBrowser
- Shows folder hierarchy
- Expandable folders
- File selection

**ImportView** (25% of Inspector):
- Bookmark management (misnamed)
- Should be "Add Folder" or "Bookmarks"
- Creates security-scoped bookmarks on iOS

### Detail View Components

**MoviesView**:
- LazyVGrid of 2:3 posters
- Still uses SwiftData @Query
- Context menu for delete
- NavigationLink to MovieView

**ShowsView**:
- Variable-size grid cards
- Still uses SwiftData @Query
- Expandable show cards
- Future: Season navigation

**MovieView**:
- AVPlayer for playback
- Fade overlay controls
- Back/Play/Fullscreen buttons
- Platform-specific volume controls

## Navigation Transitions

| From | To | Trigger | Type | Implementation |
|------|-----|---------|------|----------------|
| FileBrowser | MoviesView | Tap "Movies" folder | Detail update | onNavigate(.movies) |
| FileBrowser | ShowsView | Tap "TV Shows" folder | Detail update | onNavigate(.shows) |
| FileBrowser | Playback | Tap .m4v file | Modal/Sheet | AVPlayer(url:) |
| MoviesView | MovieView | Tap poster | Navigation push | NavigationLink |
| ImportView | Document Picker | Tap Import | Modal sheet | .fileImporter |
| Any | Fullscreen | Fullscreen button | Platform modal | .fullScreenCover |

## Platform Navigation Patterns

### iOS
```swift
// Sheet presentations
.sheet(isPresented: $showingPicker) {
    AdvancedDocumentPicker(...)
}

// Fullscreen video
.fullScreenCover(isPresented: $showFullscreen) {
    VideoPlayer(player: player)
}

// Context menus
.contextMenu {
    Button("Delete", role: .destructive) { ... }
}
```

### tvOS
```swift
// Focus-based navigation
@FocusState private var focusedItem: MediaFileItem?

// No sheets - everything inline
// No context menus - use buttons
// Remote control navigation

Button("Play") { ... }
    .focused($isPlayButtonFocused)
```

### macOS
```swift
// Native window fullscreen
NSApp.windows.first?.toggleFullScreen(nil)

// Right-click context menus
.contextMenu { ... }

// Hover states
.onHover { isHovered in ... }

// Keyboard shortcuts (future)
.keyboardShortcut(.space, modifiers: [])
```

## Empty States

### No Videos in Folder
```swift
ContentUnavailableView(
    "No Movies",
    systemImage: "film",
    description: Text("Add .m4v files to this folder")
)
```

### Nothing Selected
```swift
Text("Select an item")
    .foregroundStyle(.secondary)
```

## Navigation Issues & Improvements

### Current Issues ⚠️

1. **Dual Navigation Systems**
   - FileBrowser uses URLs
   - Detail views use SwiftData queries
   - Not synchronized

2. **Confusing Terminology**
   - "Import" doesn't import
   - Creates bookmarks instead
   - Users expect file copying

3. **State Management**
   - NavigationItem for main routing
   - URL for file selection
   - SwiftData for queries

### Proposed Improvements ✅

1. **Unified Navigation**
```swift
enum NavigationDestination {
    case folder(URL)
    case videoList(VideoKind)
    case videoPlayback(URL)
}

@State private var navigationPath: [NavigationDestination] = []
```

2. **Rename ImportView**
```swift
// Old: ImportView
// New: BookmarkManager or FolderAccess
struct BookmarkManager: View {
    // Clear naming for bookmark functionality
}
```

3. **Remove SwiftData Dependency**
```swift
// Instead of @Query
let movies = fileManager.videos(in: .movies)
let shows = fileManager.videos(in: .shows)
```

## Keyboard Navigation (Future)

### Planned Shortcuts
| Key | Action | Platform |
|-----|--------|----------|
| Space | Play/Pause | All |
| ← → | Skip ±10s | All |
| ↑ ↓ | Volume | macOS/iOS |
| F | Fullscreen | macOS |
| ⌘O | Add Folder | macOS |
| Delete | Delete Item | macOS |

## Accessibility

### Current
- Basic VoiceOver labels
- Focus indicators on tvOS
- Standard navigation announcements

### Needed
- Custom actions for complex controls
- Better descriptions for folder hierarchy
- Keyboard navigation on macOS
- Focus management in Inspector

## Performance Considerations

### Optimized ✅
- Lazy loading in grids
- Single detail view instance
- Shallow navigation stack

### Needs Work ⚠️
- Directory enumeration blocks UI
- No state restoration
- No deep linking support
- SwiftData queries unnecessary

## Testing Navigation

### Critical Paths to Test
1. Launch → Browse → Play
2. Add external folder (iOS)
3. Navigate Movies vs Shows
4. Delete from context menu
5. Fullscreen transitions
6. Platform-specific controls

### Edge Cases
- Empty folders
- Stale bookmarks (iOS)
- Disconnected drives (macOS)
- Large directories (1000+ files)
- Mixed content folders

## Navigation Roadmap

### Phase 1: Cleanup (Current)
- [x] Inspector panel implementation
- [x] FileBrowser navigation
- [ ] Rename ImportView
- [ ] Remove dual state

### Phase 2: Unification
- [ ] Single navigation model
- [ ] Remove SwiftData queries
- [ ] Filesystem-only state
- [ ] Consistent terminology

### Phase 3: Enhancement
- [ ] Keyboard navigation
- [ ] State restoration
- [ ] Deep linking
- [ ] Search/filter
- [ ] Breadcrumb trail