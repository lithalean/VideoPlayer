# 🎬 VideoPlayer

*Filesystem-First M4V Video Player for Apple Platforms*

![Platform](https://img.shields.io/badge/platform-macOS%20%7C%20iOS%20%7C%20tvOS-blue)
![Architecture](https://img.shields.io/badge/arch-Universal-green)
![Swift Version](https://img.shields.io/badge/swift-5.9+-orange)
![Status](https://img.shields.io/badge/status-v2.0%20Hybrid-yellow)

**VideoPlayer** is a modern, filesystem-based video player exclusively for M4V files. Built with SwiftUI and AVKit, it provides a DVD library aesthetic with direct file playback—no importing, no copying, just browse and play.

---

## ✨ Key Features

### 🎯 Direct Filesystem Browsing
- **No import required** - Browse and play files directly from their location
- **Security-scoped access** - iOS document picker with persistent bookmarks
- **Auto-organization** - Movies and TV Shows folders created automatically
- **Inspector panel** - Modern file browser replacing traditional sidebar
- **Platform-aware** - Full filesystem on macOS, sandbox on tvOS, bookmarks on iOS

### 📀 M4V-Exclusive Playback
- **Single format focus** - Optimized exclusively for .m4v files
- **Direct URL playback** - No file copying or duplication
- **Professional controls** - Fade overlay UI with WWDC 2025 styling
- **Platform adaptations** - Native fullscreen on each platform
- **Smooth performance** - Hardware-accelerated video decoding

### 🖼️ DVD Library Aesthetic
- **2:3 poster grids** - Professional DVD/Blu-ray aspect ratio
- **Glass materials** - Modern translucent UI elements
- **Automatic posters** - Frame extraction with caching
- **Visual organization** - Distinct Movies and TV Shows sections
- **Elegant transitions** - Smooth animations throughout

---

## 🏗️ v2.0 Architecture

### Paradigm Shift
```
OLD (v1.0): Import → Copy to Library → SwiftData → Play
NEW (v2.0): Browse Filesystem → Play In-Place → Direct Access
```

### Core Components
```
├── Core/
│   ├── FileBrowser.swift        # Media-specific browser
│   ├── FileSystem.swift         # General file operations
│   └── FileSystemModel.swift    # Platform abstractions
├── Inspector/
│   ├── InspectorPanel.swift     # 320pt glass panel
│   ├── OutlinerView.swift       # Contains FileBrowser
│   └── ImportView.swift         # Bookmark management
└── Services/
    └── VideoPlayerService.swift  # AVPlayer control
```

### Platform Experiences

#### macOS
- Full filesystem access
- Native window management
- Reveal in Finder support
- Hover states and context menus
- Traffic light window controls

#### iOS
- Document picker integration
- Security-scoped bookmarks
- Files app visibility
- Sheet-based fullscreen
- Touch-optimized controls

#### tvOS
- Sandbox-only access
- Focus-based navigation
- Remote control support
- Button-based volume
- Automatic folder creation

---

## 🚧 Current Status: Hybrid Implementation

### Working Features ✅
- Filesystem browsing across all platforms
- Direct playback without copying
- Security-scoped resources on iOS
- Inspector-based navigation
- Glass UI design system
- Movies/TV Shows organization

### Known Issues ⚠️
- **Legacy SwiftData** still running unnecessary queries
- **ImportView** misnamed (manages bookmarks, not imports)
- **Poster generation** blocks UI thread (200-500ms)
- **No file watching** for automatic updates
- **Dual navigation** state (filesystem URLs vs SwiftData)

### Migration Progress
```
✅ Phase 1: Add filesystem browsing (COMPLETE)
⚠️ Phase 2: Maintain dual system (CURRENT)
🔜 Phase 3: Remove import pipeline
🔜 Phase 4: Remove SwiftData entirely
```

---

## 🛠️ Building from Source

### Requirements
- Xcode 15.0+
- iOS 17.0+ / tvOS 17.0+ / macOS 14.0+
- Swift 5.9+

### Setup
```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/VideoPlayer.git

# Open in Xcode
cd VideoPlayer
open VideoPlayer.xcodeproj

# Build and run (⌘R)
```

### First Launch
1. App creates `Documents/VideoPlayer/` structure
2. Auto-generates `Movies/` and `TV Shows/` folders
3. On iOS: Use Import to add external folders via bookmarks
4. Place .m4v files in appropriate folders
5. Browse and play directly

---

## 🎮 Controls Reference

### Playback Controls
| Action | macOS | iOS | tvOS |
|--------|-------|-----|------|
| Play/Pause | Space / Click | Tap | Select button |
| Seek ±10s | ← → | Swipe | D-pad |
| Volume | Slider | Slider | +/- buttons |
| Fullscreen | ⌘F | Button | N/A |
| Back | Escape | Chevron | Menu |

### File Management
| Action | macOS | iOS | tvOS |
|--------|-------|-----|------|
| Browse | Direct access | Via bookmarks | Sandbox only |
| Delete | Right-click | Long press | Not available |
| Add folder | N/A | Document picker | N/A |
| Refresh | ⌘R | Pull down | Focus + Play |

---

## 📁 File Organization

### Standard Structure
```
Documents/
└── VideoPlayer/
    ├── Movies/
    │   ├── Action Movie.m4v
    │   ├── Comedy Film.m4v
    │   └── Drama Title.m4v
    └── TV Shows/
        ├── Series Name S01E01.m4v
        ├── Series Name S01E02.m4v
        └── Show Title S02E05.m4v
```

### Poster Cache
```
~/Library/Application Support/VideoPlayer/Posters/
├── [video-uuid]-poster.jpg (1200x1800)
└── Generated at 2:3 aspect ratio
```

---

## 🏗️ Technical Architecture

### Design Patterns
- **Filesystem-first** - Direct file access, no database required
- **Platform abstractions** - Unified API across iOS/tvOS/macOS
- **Security-scoped** - Proper iOS bookmark handling
- **Glass design** - Modern translucent materials
- **Singleton services** - Shared video player instance

### Performance Metrics
- **Launch time** - Under 200ms
- **Browse latency** - Instant (no database)
- **Playback start** - Direct streaming
- **Memory usage** - ~50MB baseline
- **Poster generation** - 200-500ms (needs optimization)

### Testing Coverage
- **Current** - 0% (no tests yet)
- **Priority areas**:
  - Security-scoped resources
  - Bookmark persistence
  - Platform-specific paths
  - File operations
  - Navigation state

---

## 🗺️ Development Roadmap

### Immediate (v2.1)
- [ ] Rename ImportView → BookmarkView
- [ ] Remove SwiftData queries
- [ ] Async poster generation
- [ ] File watching implementation
- [ ] Loading indicators

### Short Term (v3.0)
- [ ] Complete SwiftData removal
- [ ] Keyboard shortcuts
- [ ] Search functionality
- [ ] Sorting options
- [ ] Preview on hover

### Long Term (v4.0)
- [ ] Metadata extraction
- [ ] Watch status tracking
- [ ] Playlist support
- [ ] Chapter markers
- [ ] Subtitle selection

---

## 💡 Design Philosophy

### Why M4V Only?
Following the single-format philosophy—better to excel at one format than poorly support many. M4V provides the best balance of quality, compatibility, and features for Apple platforms.

### Why Filesystem-First?
Users already organize their files. Instead of forcing imports and creating duplicates, we browse existing structures. This eliminates storage waste and respects user organization.

### Why Inspector Panel?
Traditional sidebars can't show filesystem hierarchy effectively. The Inspector provides a dedicated space for browsing while maintaining the content area for video grids.

### Why 2:3 Posters?
DVD and Blu-ray cases use 2:3 aspect ratio. This creates instant recognition as a video library and provides optimal visual density in grids.

---

## 🤝 Contributing

### Current Priorities
1. **Testing** - Add unit tests for critical paths
2. **Performance** - Async poster generation
3. **Cleanup** - Remove legacy SwiftData code
4. **Polish** - Loading states and animations

### Code Style
- SwiftUI declarative patterns
- Platform conditionals via `#if`
- Async/await for I/O operations
- Clear separation of concerns

### Commit Guidelines
```
feat: Add new feature
fix: Bug fix
perf: Performance improvement
refactor: Code restructuring
docs: Documentation updates
test: Test additions
```

---

## 📄 License

MIT License - See [LICENSE](LICENSE) file for details

---

## 🙏 Acknowledgments

- **AVKit Team** - Professional video framework
- **SwiftUI Team** - Modern declarative UI
- **Files App Team** - Document provider infrastructure
- **QuickTime Legacy** - Inspiration for simplicity

---

## 📊 Project Health

| Metric | Status | Notes |
|--------|--------|-------|
| **Build** | ✅ Passing | All platforms |
| **Tests** | ⚠️ None | Priority for v2.1 |
| **Performance** | ⚠️ Good | Poster generation blocks UI |
| **Documentation** | ✅ Complete | Comprehensive .context/ |
| **Architecture** | ⚠️ Hybrid | Migration in progress |

---

<p align="center">
  <i>Built with SwiftUI for the Apple ecosystem. Browse, don't import.</i>
</p>

<p align="center">
  <a href="#-videoplayer">Back to top ↑</a>
</p>