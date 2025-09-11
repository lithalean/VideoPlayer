# VideoPlayer Architecture Context

**Purpose**: Technical blueprint for filesystem-first M4V video player  
**Version**: 2.0  
**Status**: Major Refactor Complete  
**Last Updated**: September 2025

## Architectural Evolution

### Version 1.0 → 2.0 Transformation
```
v1.0: Import → Copy to Library → SwiftData → Play
v2.0: Browse Filesystem → Play In-Place → Direct Access
```

The project has evolved from an import-based library system to a direct filesystem browser, eliminating file duplication while maintaining the M4V-only discipline and Movies/Shows organization.

## Core Architecture

### System Overview
```
┌─────────────────────────────────────────────┐
│             VideoPlayerApp                   │
│         (Hybrid: SwiftData + FileSystem)     │
└─────────────────┬───────────────────────────┘
                  │
    ┌─────────────┼─────────────┬──────────────┐
    ▼             ▼             ▼              ▼
┌────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│  Core  │  │Inspector │  │ Services │  │  Views   │
├────────┤  ├──────────┤  ├──────────┤  ├──────────┤
│FileBrws│  │Panel     │  │Player    │  │Movies    │
│FileSys │  │Outliner  │  │Import    │  │Shows     │
│FSModel │  │Import    │  │(legacy)  │  │MovieView │
└────────┘  └──────────┘  └──────────┘  └──────────┘
     ↕            ↕             ↕            ↕
┌─────────────────────────────────────────────┐
│         FileSystem / Documents              │
│    ├── VideoPlayer/                         │
│    │   ├── Movies/                          │
│    │   └── TV Shows/                        │
└─────────────────────────────────────────────┘
```

### Key Architectural Patterns

#### Pattern 1: Direct Filesystem Access
```
PURPOSE: Eliminate storage duplication, play files in-place
IMPLEMENTATION: 
  - FileBrowser for media-specific navigation
  - FileSystem for general file operations
  - FileSystemModel for platform abstractions
BENEFITS: No duplicate storage, instant access, real-time updates
TRADE-OFFS: Requires security-scoped resources on iOS
```

#### Pattern 2: Inspector Panel Navigation
```
PURPOSE: Replace sidebar with filesystem browser
IMPLEMENTATION:
  - InspectorPanel: Container with 75/25 split
  - OutlinerView: Contains FileBrowser
  - ImportView: Bookmark management (not file copying)
BENEFITS: Unified navigation, filesystem visibility
TRADE-OFFS: More complex than simple sidebar
```

#### Pattern 3: Platform-Specific Filesystem
```
PURPOSE: Handle iOS/tvOS/macOS filesystem differences
IMPLEMENTATION:
  iOS: Security-scoped bookmarks, document picker
  tvOS: Sandbox-only access, no imports
  macOS: Full filesystem access
BENEFITS: Native behavior per platform
TRADE-OFFS: Complex conditional compilation
```

#### Pattern 4: Hybrid Persistence
```
PURPOSE: Maintain metadata while browsing filesystem
IMPLEMENTATION:
  - SwiftData: Video metadata, posters (legacy)
  - Filesystem: Direct file access
  - Bookmarks: iOS folder persistence
BENEFITS: Fast queries, direct playback
TRADE-OFFS: Dual state management
```

## Component Architecture

### Core/ (Filesystem Layer)
```swift
FileBrowser.swift
  ├── MediaFileManager: ObservableObject
  ├── MediaFileItem: Identifiable
  └── FileBrowser: View
      └── Destination enum (movies/shows/file)

FileSystem.swift
  ├── FileSystemView: General file browser
  ├── FileTreeItemView: Recursive items
  └── CreateItemSheet: File/folder creation

FileSystemModel.swift
  ├── FileSystemItem: File/folder model
  ├── FileType: Classification system
  └── FileSystemViewModel: Platform abstractions
```

### Inspector/ (Navigation)
```swift
InspectorPanel.swift
  ├── Fixed 320pt width (macOS) / 280pt (iOS)
  ├── 75% OutlinerView / 25% ImportView split
  └── Glass design system

OutlinerView.swift
  ├── Contains FileBrowser
  └── Movies/Shows sections

ImportView.swift
  └── Bookmark management (not file import)
```

### Security Model (iOS)

#### Security-Scoped Resources
```swift
// Request access
let didAccess = url.startAccessingSecurityScopedResource()
defer { 
    if didAccess { 
        url.stopAccessingSecurityScopedResource() 
    }
}

// Persist via bookmarks
let bookmarkData = try url.bookmarkData(
    options: .minimalBookmark,
    includingResourceValuesForKeys: nil,
    relativeTo: nil
)
UserDefaults.standard.set(bookmarkData, forKey: key)
```

#### Document Picker Integration
```swift
UIDocumentPickerViewController(forOpeningContentTypes: [.folder, .item])
  → Security-scoped URL
  → Create bookmark
  → Access folder contents
```

## Data Flow

### Browse → Play Flow
```
1. FileBrowser displays Documents/VideoPlayer structure
2. User navigates Movies/ or TV Shows/ folders
3. Tap .m4v file → AVPlayer loads URL directly
4. No copying, no import, instant playback
```

### iOS Folder Access Flow
```
1. ImportView → Document Picker
2. Select folder → Security-scoped URL
3. Create bookmark → UserDefaults
4. Resolve bookmark → Access folder
5. Browse contents → Play files
```

## Platform Adaptations

### iOS
- Documents/VideoPlayer/ as root
- Security-scoped bookmarks for external folders
- Document picker for folder selection
- Sheet presentations for fullscreen
- Files app integration via Info.plist

### tvOS
- Sandbox-only (no external storage)
- Documents/VideoPlayer/ created automatically
- No import capability
- Focus-based navigation
- Button-based volume controls

### macOS
- Full filesystem access
- Home/Desktop/Documents/Downloads navigation
- Native window management
- Reveal in Finder / Open in Terminal
- Hover states on all interactive elements

## Movies/Shows Organization

### Filesystem Structure
```
Documents/
└── VideoPlayer/
    ├── Movies/
    │   ├── Action Movie.m4v
    │   └── Comedy Movie.m4v
    └── TV Shows/
        ├── Series S01E01.m4v
        └── Series S01E02.m4v
```

### Detection Logic
1. **Folder-based**: "Movies" and "TV Shows" folders
2. **Legacy SwiftData**: kindRaw field for imported videos
3. **Future**: Metadata extraction from files

## Performance Optimizations

### Current
- Direct file playback (no copying)
- Lazy loading in grids
- Poster caching in Application Support
- Single AVPlayer instance

### Needed
- Background poster generation
- File watching for updates
- Preview frame caching
- Async directory enumeration

## Security & Permissions

### Info.plist Keys
```xml
<key>UIFileSharingEnabled</key><true/>
<key>LSSupportsOpeningDocumentsInPlace</key><true/>
<key>UISupportsDocumentBrowser</key><true/>
```

### Entitlements
```xml
<key>com.apple.security.app-sandbox</key><true/>
<key>com.apple.security.files.user-selected.read-only</key><true/>
```

## Anti-Patterns to Avoid

### ❌ NEVER: Copy files on "import"
The system now browses files in-place. Never duplicate.

### ❌ NEVER: Assume filesystem access
Always use security-scoped resources on iOS.

### ❌ NEVER: Mix import and browse paradigms
The UI should be consistent: we browse, not import.

## Technical Debt & Migration

### Legacy Components (Still Present)
- VideoImportService: Partially used for metadata
- SwiftData VPVideo model: May be removed
- kindRaw workaround: Still needed if keeping SwiftData

### Migration Path
1. ✅ Phase 1: Add filesystem browsing (COMPLETE)
2. ⚠️ Phase 2: Maintain dual system (CURRENT)
3. 🔜 Phase 3: Remove import pipeline
4. 🔜 Phase 4: Optional - remove SwiftData

## Architecture Decision Records

### ADR-001: Filesystem-First Navigation
**Date**: September 2025  
**Status**: Implemented  
**Context**: Storage duplication was wasteful  
**Decision**: Browse and play files directly  
**Consequences**: More complex iOS security, simpler overall flow  

### ADR-002: Inspector Panel Design
**Date**: September 2025  
**Status**: Implemented  
**Context**: Sidebar couldn't show filesystem hierarchy  
**Decision**: Replace with Inspector containing FileBrowser  
**Consequences**: Better visibility, more complex navigation  

### ADR-003: Maintain M4V Restriction
**Date**: September 2025  
**Status**: Maintained  
**Context**: Considered expanding format support  
**Decision**: Keep M4V-only discipline  
**Consequences**: Simpler testing, consistent behavior  

### ADR-004: Hybrid SwiftData/Filesystem
**Date**: September 2025  
**Status**: Active  
**Context**: Need metadata while browsing files  
**Decision**: Keep SwiftData for now, may remove later  
**Consequences**: Dual state management complexity