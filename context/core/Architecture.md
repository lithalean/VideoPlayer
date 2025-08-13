# VideoPlayer Architecture Context

**Purpose**: Technical blueprint for M4V-only video player forked from AudioPlayer  
**Version**: 1.0  
**Status**: Production Ready  
**Last Updated**: August 12, 2025

## Key Concepts (AI Quick Reference)

### Core Architecture Pattern
```
M4V Files + AVPlayer + SwiftData = QuickTime-style Player
AudioPlayer Fork → Video Transformation → DVD Library Experience
```

### Critical Design Elements
1. **M4V Exclusivity** - Single format discipline (like AudioPlayer's M4A-only)
2. **2:3 Poster Ratio** - DVD box art aesthetic (replaces square album art)
3. **Movies/Shows Split** - VideoKind enum with kindRaw workaround
4. **Platform Adaptations** - tvOS no-drag, macOS fullscreen, iOS sheets

## System Architecture

### Core Design Pattern
**Forked Audio → Video Architecture**
- AVAudioPlayer replaced with AVPlayer for video playback
- Album/Song models transformed to VPVideo with VideoKind
- Square album artwork becomes 2:3 movie posters
- Music library metaphor adapted to video library (Movies/Shows)

### Component Architecture
```
┌─────────────────────────────────────────┐
│          VideoPlayerApp                  │
│         (SwiftData Container)            │
└─────────────┬───────────────────────────┘
              │
    ┌─────────┴──────────┬──────────────┐
    ▼                    ▼              ▼
┌─────────┐      ┌──────────┐    ┌──────────┐
│ Models  │      │ Services │    │  Views   │
├─────────┤      ├──────────┤    ├──────────┤
│ VPVideo │◄────►│Import    │    │ContentView│
│VideoKind│      │Player    │◄───│MovieView │
└─────────┘      └──────────┘    │MoviesView│
                                  └──────────┘
```

## Technical Architecture

### State Management
- **SwiftData**: VPVideo model with @Model macro
- **@Query**: String-based predicates using kindRaw to avoid enum issues
- **@State/@Published**: View-level state for playback controls
- **Singleton Services**: VideoPlayerService.shared pattern

### Platform Divergence
```swift
#if os(tvOS)
    // Focus engine navigation, button-based volume
    // No file import, no drag gestures
    // Larger UI elements for TV viewing
#elseif os(iOS) 
    // fileImporter, sheet presentations, drag gestures
    // NavigationSplitView with sidebar
    // Context menus for actions
#else // macOS
    // Window fullscreen, sidebar with traffic light spacing
    // Hover states, native window management
    // Full keyboard and mouse support
#endif
```

### Data Flow Architecture
```
User Action → View → Service → Model → SwiftData
                ↓        ↓
            AVPlayer  FileSystem
```

## Design Patterns

### Pattern 1: Format Exclusivity
```
PURPOSE: Reduce complexity and edge cases
IMPLEMENTATION: .mpeg4Movie UTType validation, .m4v extension check
BENEFITS: Stable, predictable behavior (inherited from AudioPlayer)
TRADE-OFFS: Users must convert other formats
```

### Pattern 2: kindRaw String Workaround
```
PURPOSE: SwiftData predicates fail with enum.rawValue
IMPLEMENTATION: Store string primitive, wrap with computed property
BENEFITS: Reliable queries across all platforms
TRADE-OFFS: Dual storage (enum + string)
```

### Pattern 3: Service Layer Singleton
```
PURPOSE: Centralized video playback and import logic
IMPLEMENTATION: VideoPlayerService.shared, VideoImportService.shared
BENEFITS: Single source of truth, easy DI in views
TRADE-OFFS: Global state, testing complexity
```

### Pattern 4: Coordinated File Access
```
PURPOSE: Prevent file access crashes during import
IMPLEMENTATION: NSFileCoordinator for all file operations
BENEFITS: Thread-safe, system-coordinated access
TRADE-OFFS: Slightly more complex than direct access
```

## Anti-Patterns to Avoid

### ❌ NEVER: Query enum values directly in predicates
```swift
// WRONG - This crashes at runtime
@Query(filter: #Predicate<VPVideo> { $0.kind == .movie })

// CORRECT - Use kindRaw string
@Query(filter: #Predicate<VPVideo> { $0.kindRaw == "movie" })
```

### ❌ NEVER: Use DragGesture on tvOS
```swift
// WRONG - tvOS doesn't support drag
.gesture(DragGesture()...)

// CORRECT - Platform-specific controls
#if os(tvOS)
    Button("-") { adjustVolume(-0.1) }
    Text("\(Int(volume * 100))%")
    Button("+") { adjustVolume(0.1) }
#else
    GlassVolumeSlider(volume: $volume)
#endif
```

### ❌ NEVER: Access files without coordination
```swift
// WRONG - Can cause overlapping access
try FileManager.default.copyItem(at: source, to: dest)

// CORRECT - Use coordinator
coordinator.coordinate(writingItemAt: source, options: .forMoving) { url in
    try FileManager.default.copyItem(at: url, to: dest)
}
```

## Architectural Decisions Log

### Decision: Fork AudioPlayer → VideoPlayer
**Rationale**: Proven architecture, similar single-format philosophy  
**Implementation**: Replace audio components with video equivalents  
**Result**: Rapid development with stable foundation

### Decision: 2:3 Poster Aspect Ratio
**Rationale**: Match DVD/Blu-ray box art standard  
**Implementation**: AspectRatio(2/3) throughout poster views  
**Result**: Professional, recognizable video library aesthetic

### Decision: Singleton Services
**Rationale**: Single AVPlayer instance, centralized import logic  
**Implementation**: Static shared instances with @MainActor  
**Result**: Clean dependency injection, predictable state

### Decision: SwiftData for Persistence
**Rationale**: Native Apple solution, good SwiftUI integration  
**Implementation**: @Model macro with manual migration support  
**Result**: Simple persistence with some macro limitations

## Component Responsibilities

### Models (VPVideo)
- Store video metadata
- Provide kindRaw for queries
- Define relationships (future: playlists)

### Services
- **VideoImportService**: File validation, import, deletion
- **VideoPlayerService**: AVPlayer management, playback state

### Views
- **ContentView**: Platform-specific navigation container
- **Sidebar**: Library organization, import controls
- **MoviesView/ShowsView**: Grid displays
- **MovieView**: Playback screen with controls
- **MoviePoster**: 2:3 aspect ratio component

## Performance Considerations

### Optimized
- LazyVGrid for efficient scrolling
- Poster caching via filesystem
- Single AVPlayer instance
- SwiftData lazy loading

### Needs Optimization
- Poster generation blocks UI
- No preview frame caching
- Full file copy on import
- No background queue usage

## Security & Sandboxing

### Entitlements
- `com.apple.security.app-sandbox`: YES
- `com.apple.security.files.user-selected.read-only`: YES

### File Access
- Import: Security-scoped via fileImporter
- Playback: Direct access to app container
- Deletion: Coordinated removal

## Future Architecture Considerations

### Scalability
- Move to actor model for services
- Background queue for heavy operations
- Streaming instead of full copy
- CloudKit sync preparation

### Modularity
- Separate playback engine
- Plugin architecture for formats
- Themeable UI system
- Testable service layer