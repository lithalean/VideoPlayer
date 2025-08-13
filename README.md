# VideoPlayer
*A Modern QuickTime-Style M4V Video Player for Apple Platforms*

![Platform Support](https://img.shields.io/badge/platform-macOS%20%7C%20iOS%20%7C%20iPadOS%20%7C%20tvOS-blue)
![Swift Version](https://img.shields.io/badge/swift-5.9+-orange)
![iOS Version](https://img.shields.io/badge/iOS-17.0+-green)
![License](https://img.shields.io/badge/license-MIT-blue)

## ✨ Current Status: **PRODUCTION READY**

Fully functional M4V video player with **2:3 poster grids**, **fade overlay playback**, and **platform-specific adaptations**. Forked from AudioPlayer for proven architecture. 🎬

## 🎯 Project Vision

VideoPlayer delivers a focused, elegant video playback experience exclusively for M4V files. Following the single-format philosophy, it provides a **DVD library aesthetic** with professional 2:3 poster grids that scale beautifully across **iOS, tvOS, and macOS**.

## 🚀 Features

### 🎬 Video Playback
- ✅ **M4V-exclusive** format support (.mpeg4Movie)
- ✅ Smooth playback via **AVPlayer** with AVKit integration
- ✅ **Fade overlay UI** - controls disappear during playback
- ✅ Platform-specific **fullscreen modes**
- ✅ Back and fullscreen navigation controls

### 📚 Video Library
- ✅ **Movies & Shows** organization with VideoKind enum
- ✅ **2:3 poster grids** matching DVD/Blu-ray standards
- ✅ **Dynamic badge counts** in sidebar navigation
- ✅ Automatic **poster generation** from video frames
- ✅ **File import** with kind selection (iOS/macOS)

### 🖥️ Platform Adaptations
- ✅ **iOS** – NavigationSplitView, sheet fullscreen, drag gestures
- ✅ **tvOS** – Focus engine, button controls, larger UI (no drag)
- ✅ **macOS** – Window fullscreen, sidebar with traffic lights
- ✅ **SwiftData** persistence with kindRaw workaround

### 🎨 Visual Design
- ✅ **DVD library aesthetic** with 2:3 poster aspect ratio
- ✅ **Glass materials** and gradient backdrops
- ✅ **WWDC 2025-style** playback controls
- ✅ Bottom fade overlay on posters
- ✅ Platform-specific shadows and corner radii

## 🛠 Technical Overview

### Architecture
- **SwiftUI + SwiftData** – Modern declarative UI with persistence
- **AVKit + AVFoundation** – Professional video playback
- **Singleton Services** – VideoImportService & VideoPlayerService
- **Platform Conditionals** – 58 #if blocks for platform-specific code

### Key Components
- **VPVideo Model** – SwiftData model with kindRaw workaround
- **Coordinated File Import** – NSFileCoordinator for safe operations
- **Poster System** – JPEG cache at 1200x1800 (2:3 ratio)
- **Fade Animations** – 250ms transitions for overlay elements

### Origin Story
Forked from AudioPlayer to leverage:
- Proven SwiftData patterns
- Platform-specific adaptations
- Single-format discipline
- Service layer architecture

## 📦 Installation & Setup

### Prerequisites
- **Xcode 15.0+**
- **iOS 17.0+, tvOS 17.0+, macOS 14.0+**
- **Swift 5.9+**
- Optimized for **Apple Silicon**

### Run the Project
1. Clone the repository:
   ```bash
   git clone https://github.com/YOUR_USERNAME/VideoPlayer.git
   ```
2. Open `VideoPlayer.xcodeproj` in Xcode
3. Build & run on your preferred platform

### Import Videos
1. Click import controls (bottom of sidebar)
2. Select Movie or Show kind
3. Choose .m4v files only
4. Videos import with poster generation

## ⌨️ Controls

### iOS/iPadOS/macOS
- **Play** – Tap play button (overlay fades)
- **Back** – Top-left chevron button
- **Fullscreen** – Top-right expand button
- **Volume** – Drag slider (iOS/macOS)
- **Import** – File picker for .m4v files

### tvOS
- **Play/Pause** – Select button on remote
- **Volume** – Plus/minus buttons (no drag)
- **Navigation** – Focus-based with remote
- **Back** – Menu button
- **Note**: No file import on tvOS

## 🗂 Project Structure

```
VideoPlayer/
├── App/
│   ├── VideoPlayerApp.swift      # Entry point
│   └── Assets.xcassets           # App resources
├── Models/
│   └── VideoModels.swift         # VPVideo with kindRaw
├── Services/
│   ├── VideoImportService.swift  # Import & deletion
│   └── VideoPlayerService.swift  # Playback control
└── Views/
    ├── ContentView.swift         # Main navigation
    ├── Sidebar.swift            # Library navigation
    ├── MoviesView.swift         # Movie grid (2:3)
    ├── ShowsView.swift          # Shows grid
    ├── MovieView.swift          # Playback screen
    ├── MoviePoster.swift        # 2:3 poster component
    └── VideoPlaybackBar.swift   # WWDC25 controls
```

## 🐛 Known Issues

- **Poster Orphaning** – Deleted videos leave poster files
- **UI Blocking** – Poster generation blocks main thread (200-500ms)
- **No Scrubbing** – Preview scrubbing not implemented
- **No Subtitles** – Subtitle tracks not selectable

## 🗺 Development Roadmap

### ✅ Phase 1: Core (Complete)
- M4V import pipeline
- Movies/Shows organization  
- Basic playback with fade overlay
- 2:3 poster generation

### 🚧 Phase 2: Polish (Current)
- Fix poster orphaning
- Background poster generation
- Keyboard shortcuts
- Volume persistence

### 🔮 Phase 3: Enhanced (Future)
- Preview scrubbing bar
- Picture-in-picture
- AirPlay integration
- Playlist/queue system
- Chapter markers
- Subtitle support

## 🏗 Technical Decisions

### Why M4V Only?
Following AudioPlayer's M4A-only philosophy - better to support one format perfectly than many poorly.

### Why 2:3 Posters?
DVD/Blu-ray standard creates instant recognition as a video library.

### Why kindRaw?
SwiftData can't filter enums in predicates - string workaround enables queries.

### Why Fork AudioPlayer?
60% pattern reuse, proven architecture, 2 days vs 2 weeks development.

## 📄 License

MIT License - See LICENSE file for details

## 🙏 Acknowledgments

- Forked from **AudioPlayer** for architecture
- Inspired by **QuickTime Player** for simplicity
- **WWDC 2025** design principles for modern UI
- **DVD library** aesthetic for visual identity

---

*Built with SwiftUI, SwiftData, and a focus on elegant simplicity.*