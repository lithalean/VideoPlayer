# VideoPlayer Context Engineering Manifest

**Purpose**: Master reference for AI systems to understand the VideoPlayer v2.0 structure  
**Last Updated**: September 2025  
**Manifest Version**: 2.0  
**Project Status**: Hybrid Implementation (Filesystem + Legacy SwiftData)

## Architecture Summary

**v2.0 Key Change**: Direct filesystem browsing replaces import/copy paradigm
```
OLD: Import files → Copy to library → SwiftData → Play
NEW: Browse filesystem → Play in-place → No copying
```

## Module Version Registry

| Module | Version | Status | Size | Description |
|--------|---------|--------|------|-------------|
| ARCHITECTURE.md | 2.0 | ✅ Current | 10KB | Filesystem-first design |
| FILESYSTEM.md | 1.0 | ✅ New | 14KB | Deep filesystem implementation |
| IMPLEMENTATION.md | 2.0 | ✅ Current | 12KB | Hybrid state documentation |
| NAVIGATION.md | 2.0 | ✅ Current | 11KB | Inspector-based flow |
| VISUAL.md | 2.0 | ✅ Current | 13KB | Glass design system |
| INTEGRATION.yaml | 1.0 | ✅ New | 9KB | Dependencies & integrations |
| EVOLUTION.md | 1.0 | 📝 Not Updated | 10KB | Legacy - needs v2.0 update |
| session.json | - | 📝 Not Updated | 3KB | Legacy - needs v2.0 update |

## Quick Reference Matrix

| Information Needed | Primary Module | Secondary Module | Notes |
|-------------------|----------------|------------------|-------|
| **Filesystem browsing** | FILESYSTEM.md | IMPLEMENTATION.md | Core v2.0 feature |
| **Security-scoped resources** | FILESYSTEM.md | INTEGRATION.yaml | iOS critical |
| **Inspector panel** | NAVIGATION.md | VISUAL.md | Replaces Sidebar |
| **Current state** | IMPLEMENTATION.md | - | Shows hybrid issues |
| **Glass design** | VISUAL.md | - | WWDC 2025 style |
| **Platform differences** | FILESYSTEM.md | ARCHITECTURE.md | iOS/tvOS/macOS |
| **SwiftData status** | IMPLEMENTATION.md | INTEGRATION.yaml | Marked for removal |
| **Dependencies** | INTEGRATION.yaml | - | All frameworks |
| **Import → Browse migration** | ARCHITECTURE.md | IMPLEMENTATION.md | Paradigm shift |
| **Movies/Shows organization** | FILESYSTEM.md | NAVIGATION.md | Folder-based |

## Context Loading Strategy

### For Understanding v2.0 Architecture
1. ARCHITECTURE.md (paradigm shift)
2. FILESYSTEM.md (new core system)
3. IMPLEMENTATION.md (hybrid state)

### For Filesystem Work
1. FILESYSTEM.md (complete guide)
2. INTEGRATION.yaml (security details)
3. IMPLEMENTATION.md (current issues)

### For UI/Navigation
1. NAVIGATION.md (Inspector flow)
2. VISUAL.md (glass design)
3. IMPLEMENTATION.md (component state)

### For Platform-Specific Work
1. FILESYSTEM.md (platform sections)
2. INTEGRATION.yaml (platform requirements)
3. ARCHITECTURE.md (adaptations)

### For Cleanup/Refactoring
1. IMPLEMENTATION.md (technical debt)
2. INTEGRATION.yaml (deprecation timeline)
3. ARCHITECTURE.md (anti-patterns)

## Key Project Facts

- **Version**: 2.0 (Hybrid filesystem + legacy SwiftData)
- **Format**: M4V-only video player (unchanged)
- **Architecture**: Direct filesystem browsing (NEW)
- **Navigation**: Inspector panel replaces Sidebar (NEW)
- **Platforms**: iOS, tvOS, macOS (fully native)
- **Framework**: SwiftUI + AVKit + FileManager
- **Design**: Glass UI + 2:3 DVD posters
- **Status**: Working but needs cleanup

## Critical v2.0 Components

### New Core Components
```
Core/
├── FileBrowser.swift      # Media-specific browser
├── FileSystem.swift       # General file operations  
└── FileSystemModel.swift  # Platform abstractions

Inspector/
├── InspectorPanel.swift   # Replaces Sidebar
├── OutlinerView.swift     # Contains FileBrowser
└── ImportView.swift       # Misnamed - manages bookmarks
```

### Legacy Components (Still Active)
```
Services/
├── VideoImportService.swift  # Partially used
└── VideoPlayerService.swift  # Still needed

Models/
└── VideoModels.swift         # SwiftData - to be removed
```

## Module Health Indicators

- ✅ **Green**: Module is current and accurate
- ⚠️ **Yellow**: Module works but has issues
- 📝 **Gray**: Module needs update for v2.0
- ❌ **Red**: Module is broken/obsolete

## Critical Issues to Address

1. **SwiftData Still Active** - Queries running despite filesystem browsing
2. **ImportView Misnamed** - Actually manages bookmarks, not imports
3. **Dual Navigation** - FileBrowser URLs vs SwiftData queries
4. **Poster Generation** - Still blocks UI thread
5. **No File Watching** - Manual refresh required

## Deprecation Timeline

### v2.1 (Immediate)
- Rename ImportView → BookmarkView
- Remove import terminology from UI

### v3.0 (Next Major)
- Remove SwiftData completely
- Remove VideoImportService
- Unify navigation state

### v4.0 (Future)
- Async poster generation
- File watching implementation

## Platform-Specific Critical Paths

### iOS
1. Document picker → Security-scoped URL
2. Create bookmark → UserDefaults
3. Browse folders → Play files

### tvOS
1. Sandbox only - no external access
2. Auto-created Movies/TV Shows folders
3. Focus-based navigation

### macOS
1. Full filesystem access
2. Reveal in Finder / Open in Terminal
3. Native window management

## Known Cross-Module Dependencies

- IMPLEMENTATION depends on all modules for current state
- NAVIGATION depends on FILESYSTEM for Inspector content
- VISUAL depends on NAVIGATION for Inspector design
- FILESYSTEM provides core for all UI components
- INTEGRATION documents all external requirements
- ARCHITECTURE explains the paradigm shift

## AI Instructions for Using This v2.0 System

1. **First Load**: Read MANIFEST.md to understand v2.0 changes
2. **Architecture**: Load ARCHITECTURE.md for paradigm shift
3. **New System**: Load FILESYSTEM.md for core implementation
4. **Current State**: Load IMPLEMENTATION.md for hybrid issues
5. **Navigation**: Load NAVIGATION.md for Inspector flow
6. **Cleanup**: Check INTEGRATION.yaml for deprecation timeline

## Testing Priority Areas

1. Security-scoped resources (iOS)
2. Bookmark persistence
3. Direct file playback
4. Inspector navigation
5. Platform-specific paths

## Performance Metrics

- **Good**: Direct playback (no copying)
- **Bad**: Poster generation blocks UI
- **Bad**: SwiftData queries unnecessary
- **Needed**: Async directory operations

## Module File Locations

```
VideoPlayer/
├── .context/
│   ├── MANIFEST.md (this file)
│   ├── core/
│   │   ├── ARCHITECTURE.md (v2.0)
│   │   ├── IMPLEMENTATION.md (v2.0)
│   │   ├── VISUAL.md (v2.0)
│   │   ├── NAVIGATION.md (v2.0)
│   │   └── FILESYSTEM.md (NEW)
│   └── dynamic/
│       ├── INTEGRATION.yaml (NEW)
│       ├── EVOLUTION.md (needs update)
│       └── session.json (needs update)
```

## Quick Health Check

**Overall Health**: 75/100

**Working Well**:
- ✅ Filesystem browsing functional
- ✅ Direct playback eliminates storage issues
- ✅ Platform abstractions solid
- ✅ Security model correct

**Needs Attention**:
- ⚠️ Hybrid state confusing
- ⚠️ Legacy components active
- ⚠️ ImportView naming
- ⚠️ Performance issues
- ⚠️ No tests

## Recommended Next Actions

1. **Read IMPLEMENTATION.md** to understand hybrid state
2. **Check deprecation timeline** in INTEGRATION.yaml
3. **Rename ImportView** to match actual function
4. **Plan SwiftData removal** using ARCHITECTURE.md guidance
5. **Add tests** for filesystem operations