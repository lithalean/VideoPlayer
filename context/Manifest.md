# VideoPlayer Context Engineering Manifest

**Purpose**: Master reference for AI systems to understand the VideoPlayer context module structure  
**Last Updated**: August 12, 2025  
**Manifest Version**: 1.0  
**Project Origin**: Forked from AudioPlayer

## Module Version Registry

| Module | Version | Last Modified | Size | Health |
|--------|---------|--------------|------|---------|
| ARCHITECTURE.md | 1.0 | 2025-08-12 | 8KB | ✅ |
| IMPLEMENTATION.md | 1.0 | 2025-08-12 | 12KB | ✅ |
| VISUAL.md | 1.0 | 2025-08-12 | 9KB | ✅ |
| NAVIGATION.md | 1.0 | 2025-08-12 | 7KB | ✅ |
| INTEGRATION.yaml | 1.0 | 2025-08-12 | 4KB | ✅ |
| EVOLUTION.md | 1.0 | 2025-08-12 | 10KB | ✅ |
| session.json | - | DYNAMIC | 3KB | ✅ |

## Quick Reference Matrix

| Information Needed | Primary Module | Secondary Module |
|-------------------|----------------|------------------|
| How video playback works | ARCHITECTURE.md | IMPLEMENTATION.md |
| Current working features | IMPLEMENTATION.md | session.json |
| Visual poster design | VISUAL.md | IMPLEMENTATION.md |
| Movies/Shows navigation | NAVIGATION.md | IMPLEMENTATION.md |
| AVKit/SwiftData usage | INTEGRATION.yaml | ARCHITECTURE.md |
| Fork from AudioPlayer | EVOLUTION.md | ARCHITECTURE.md |
| Recent development | session.json | EVOLUTION.md |
| kindRaw workaround | ARCHITECTURE.md | EVOLUTION.md |
| Platform differences | IMPLEMENTATION.md | ARCHITECTURE.md |

## Context Loading Strategy

### For Video Playback Issues
1. IMPLEMENTATION.md (AVPlayer integration)
2. ARCHITECTURE.md (playback patterns)
3. INTEGRATION.yaml (AVKit details)

### For UI/UX Work
1. VISUAL.md (2:3 poster design)
2. NAVIGATION.md (Movies/Shows flow)
3. IMPLEMENTATION.md (view components)

### For Platform-Specific Features
1. ARCHITECTURE.md (platform patterns)
2. IMPLEMENTATION.md (platform code blocks)
3. EVOLUTION.md (tvOS adaptations)

### For Bug Fixes
1. IMPLEMENTATION.md (current state)
2. session.json (recent changes)
3. ARCHITECTURE.md (design constraints)

### For Understanding the Fork
1. EVOLUTION.md (fork history)
2. ARCHITECTURE.md (transformed patterns)
3. IMPLEMENTATION.md (what changed)

## Key Project Facts
- **Format**: M4V-only video player (mpeg4Movie UTType)
- **Origin**: Forked from AudioPlayer music app
- **Platforms**: iOS, tvOS, macOS (fully native)
- **Framework**: SwiftUI + SwiftData + AVKit
- **Design**: QuickTime-style with 2:3 DVD poster aesthetic
- **Architecture**: Singleton services, SwiftData persistence
- **Special**: kindRaw workaround for SwiftData enum predicates

## Module Health Indicators
- ✅ **Green**: Module is complete and accurate
- 🔄 **Yellow**: Module needs minor updates
- ❌ **Red**: Module is outdated or broken
- 📝 **Gray**: Module is placeholder/incomplete

## Critical Patterns to Preserve
1. **M4V Format Exclusivity** - No other formats, period
2. **2:3 Poster Aspect Ratio** - DVD box art standard
3. **kindRaw String Workaround** - Required for SwiftData queries
4. **Platform-Specific Controls** - tvOS buttons, iOS/macOS drag
5. **Fade Overlay Playback** - UI disappears during playback

## Known Cross-Module Dependencies
- IMPLEMENTATION depends on ARCHITECTURE for patterns
- VISUAL depends on IMPLEMENTATION for actual components
- NAVIGATION depends on IMPLEMENTATION for view names
- EVOLUTION explains decisions in ARCHITECTURE
- session.json tracks changes across all modules
- INTEGRATION.yaml lists all external dependencies

## AI Instructions for Using This System
1. **First Load**: Always read MANIFEST.md first
2. **Understand Design**: Load ARCHITECTURE.md for patterns
3. **Check Status**: Load IMPLEMENTATION.md for current state
4. **For UI Work**: Load VISUAL.md and NAVIGATION.md
5. **For History**: Review EVOLUTION.md for context
6. **Continue Work**: Check session.json for recent activity

## Module File Locations
```
VideoPlayer/
├── .context/
│   ├── MANIFEST.md (this file)
│   ├── core/
│   │   ├── ARCHITECTURE.md
│   │   ├── IMPLEMENTATION.md
│   │   ├── VISUAL.md
│   │   └── NAVIGATION.md
│   └── dynamic/
│       ├── INTEGRATION.yaml
│       ├── EVOLUTION.md
│       └── session.json
```