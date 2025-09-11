# VideoPlayer Visual Design Context

**Purpose**: Visual design system for filesystem-based video player with glass UI  
**Version**: 2.0  
**Design Language**: Apple HIG + Glass Design + DVD Library Aesthetic  
**Last Updated**: September 2025

## Visual Architecture

### Design System Hierarchy
```
┌─────────────────────────────────────┐
│      Glass Design System (2025)     │
│  ┌────────────────────────────────┐ │
│  │    Inspector Panel (Glass)     │ │
│  │  ┌──────────────────────────┐  │ │
│  │  │   FileBrowser (Lists)    │  │ │
│  │  └──────────────────────────┘  │ │
│  └────────────────────────────────┘ │
│  ┌────────────────────────────────┐ │
│  │    Content Grids (2:3)         │ │
│  │  ┌────┐ ┌────┐ ┌────┐         │ │
│  │  │    │ │    │ │    │         │ │
│  │  └────┘ └────┘ └────┘         │ │
│  └────────────────────────────────┘ │
└─────────────────────────────────────┘
```

## Core Visual Components

### Inspector Panel (Glass Design)

```yaml
dimensions:
  width: 
    macOS: 320pt
    iOS: 280pt
    tvOS: 360pt (standalone)
  split: 
    outliner: 75%
    import: 25%

glass_style:
  material: .regularMaterial
  corner_radius: 16pt (continuous)
  shadow:
    color: black(0.15)
    radius: 20pt
    offset: { x: 0, y: 8 }
  border:
    gradient: [white(0.2), white(0.05)]
    width: 0.5pt

internal_spacing:
  padding: 16pt
  section_gap: 0pt (divider separates)
  content_inset: 16pt
```

### FileBrowser Visual Design

```yaml
list_style:
  row_height: 
    compact: 32pt
    regular: 44pt
  indentation: 20pt per level
  icon_size: 
    folder: 16pt
    file: 14pt
  spacing: 
    icon_to_text: 8pt
    horizontal_padding: 12pt
    vertical_padding: 6pt

folder_colors:
  movies: purple
  tv_shows: blue
  default: secondary

selection_style:
  background: accentColor(0.15)
  corner_radius: 6pt
  animation: easeInOut(0.2s)

hover_style:
  background: secondary(0.05)
  show_actions: true (ellipsis menu)
```

### Glass Button Style (Shared)

```swift
struct GlassButtonStyle: ButtonStyle {
    // Ultra-thin material background
    background: .ultraThinMaterial
    corner_radius: 10pt (continuous)
    padding: { h: 12pt, v: 8pt }
    border: primary(0.08)
    
    // Pressed state
    scale: 0.97
    animation: easeOut(0.08s)
}
```

## Color System

### System Colors (Adaptive)

| Element | Light Mode | Dark Mode | Usage |
|---------|------------|-----------|-------|
| **Inspector BG** | .systemBackground | black(0.95) | Panel background |
| **Glass Material** | .regularMaterial | .ultraThinMaterial | Controls/overlays |
| **List Selection** | .accentColor(0.2) | .accentColor(0.15) | Selected items |
| **Hover State** | .secondary(0.1) | .secondary(0.05) | Mouse hover |
| **Divider** | .separator | .separator | Section dividers |
| **Icon Default** | .secondary | .secondary | Unselected icons |
| **Folder Colors** | | | |
| - Movies | .purple | .purple | Movies folder |
| - TV Shows | .blue | .blue | TV Shows folder |
| - Default | .secondary | .secondary | Regular folders |

### Movie Poster Colors (Unchanged)

```yaml
poster_gradient:
  fallback:
    colors: [accentColor(0.55), accentColor(0.25), black(0.25)]
    direction: topLeading → bottomTrailing
  
bottom_fade:
  stops:
    - { color: clear, location: 0.60 }
    - { color: black(0.17), location: 0.82 }
    - { color: black(0.28), location: 1.00 }

duration_pill:
  background: black(0.55)
  text: white
  padding: { h: 6pt, v: 3pt }
```

## Typography System

### Inspector Typography

| Style | Size | Weight | Usage |
|-------|------|--------|-------|
| **Panel Title** | .headline | .semibold | "Video Library" |
| **Folder Name** | .system(13) | .regular | File/folder labels |
| **File Info** | .system(11) | .regular | Size, date |
| **Empty State** | .subheadline | .regular | "No videos found" |
| **Button Label** | .system(14) | .semibold | Import/Create |

### Grid Typography (Unchanged)

| Style | iOS/macOS | tvOS | Weight | Usage |
|-------|-----------|------|--------|-------|
| **Grid Title** | .subheadline | .headline | .semibold | Poster labels |
| **Duration** | .caption2 | .caption2 | .regular | Time pills |
| **Empty Message** | .body | .title3 | .regular | No content |

## Spacing & Layout

### Inspector Layout

```yaml
inspector_panel:
  total_width: { macOS: 320pt, iOS: 280pt }
  content_sections:
    outliner:
      height: 75%
      padding: 16pt
      content: FileBrowser
    
    import:
      height: 25%
      padding: 8pt
      background: .secondary(0.05)
      content: ImportView (bookmarks)

file_browser:
  hierarchy:
    root_indent: 0pt
    child_indent: 20pt per level
    max_depth: unlimited
  
  row_layout:
    [indent] [chevron:16] [icon:20] [8pt] [name:flex] [info] [menu:20]
```

### Grid Layouts (Unchanged)

```yaml
movies_grid:
  type: LazyVGrid
  columns: adaptive(min: 140pt, max: 220pt)
  spacing: 16pt
  poster_aspect: 2:3

shows_grid:
  type: LazyVGrid
  pattern: [large, medium, medium, small, small, small]
  spacing: { h: 16pt, v: 16pt }
```

## Animation Specifications

### Inspector Animations

```yaml
folder_expand:
  trigger: chevron tap
  animation: spring(response: 0.3, damping: 0.8)
  chevron_rotation: 0° → 90°

row_selection:
  duration: 200ms
  curve: easeInOut
  background: transparent → accentColor(0.15)

hover_state:
  duration: 150ms
  show_menu: true
  background: transparent → secondary(0.05)

import_sheet:
  presentation: sheet
  animation: spring(response: 0.45, damping: 0.9)
```

### Playback Animations (Unchanged)

```yaml
overlay_fade:
  trigger: play state change
  duration: 250ms
  opacity: 1.0 → 0.0
  curve: easeInOut
```

## Platform Visual Adaptations

### iOS

```yaml
inspector:
  width: 280pt
  material: .regularMaterial
  safe_area: respected

navigation:
  style: NavigationSplitView
  column_visibility: automatic

sheets:
  document_picker: .pageSheet
  fullscreen_video: .fullScreenCover

gestures:
  context_menu: long press
  selection: tap
  expand: tap chevron
```

### tvOS

```yaml
inspector:
  merged_with_content: true
  max_height: 360pt
  focus_indicators: enhanced

focus_effects:
  scale: 1.08
  animation: 200ms
  shadow: enhanced

navigation:
  style: NavigationStack
  remote_control: directional
```

### macOS

```yaml
inspector:
  width: 320pt
  traffic_light_spacing: 28pt
  material: .regularMaterial

window:
  style: native
  fullscreen: native toggle
  toolbar: unified

interactions:
  hover: enabled
  right_click: context menu
  cursor: changes on hover
```

## Glass Design System Details

### Material Hierarchy

```yaml
levels:
  base: 
    - .regularMaterial (Inspector panel)
  overlay_1:
    - .ultraThinMaterial (Controls, buttons)
  overlay_2:
    - .thinMaterial (Popovers, menus)
  overlay_3:
    - .thickMaterial (Modals, sheets)
```

### Border & Shadow System

```yaml
borders:
  standard:
    gradient: [white(0.2) top, white(0.05) bottom]
    width: 0.5pt
    corner_radius: continuous
  
  selected:
    color: accentColor
    width: 2pt
    corner_radius: continuous

shadows:
  ambient:
    color: black(0.1)
    radius: 10pt
    offset: { x: 0, y: 2 }
  
  key_light:
    color: black(0.15)
    radius: 20pt
    offset: { x: 0, y: 8 }
  
  focus_ring:
    color: accentColor(0.4)
    radius: 4pt
    offset: { x: 0, y: 0 }
```

## Icon System

### SF Symbols Usage

| Icon | Symbol | Size | Context |
|------|--------|------|---------|
| **Folder** | folder.fill | 16pt | Directory |
| **Movies** | film.fill | 16pt | Movies folder |
| **TV Shows** | tv.fill | 16pt | Shows folder |
| **Video** | play.rectangle.fill | 14pt | Video file |
| **Chevron** | chevron.right/down | 10pt | Expand/collapse |
| **Menu** | ellipsis.circle | 12pt | Context menu |
| **Import** | square.and.arrow.down | 16pt | Add folder |
| **Create** | plus.circle | 14pt | New item |

### Icon Colors

```yaml
folders:
  movies: purple
  tv_shows: blue
  default: secondary

files:
  video: green
  other: secondary

states:
  selected: white (on accent)
  hover: primary
  default: secondary
```

## Visual Polish Details

### Corner Radius System

| Component | Radius | Style |
|-----------|--------|-------|
| **Inspector Panel** | 16pt | continuous |
| **List Selection** | 6pt | continuous |
| **Buttons** | 10pt | continuous |
| **Posters** | 12pt | continuous |
| **Pills** | full | capsule |
| **Sheets** | 20pt | continuous |

### Blur Effects

```yaml
glass_blur:
  ultra_thin: 20pt radius
  thin: 30pt radius
  regular: 40pt radius
  thick: 50pt radius
  
background_blur:
  video_overlay: 60pt
  sheet_backdrop: 80pt
```

## Accessibility Visual Design

### Contrast Requirements

- Text on glass: 7:1 minimum
- Icons on glass: 4.5:1 minimum
- Focus indicators: 3:1 minimum
- Selection states: clearly visible

### Focus Indicators

```yaml
keyboard_focus:
  style: ring
  color: accentColor
  width: 2pt
  offset: 2pt

voiceover_focus:
  style: bold ring
  color: white
  width: 3pt
  background: black(0.8)
```

### Reduce Motion

When enabled:
- Instant transitions (no animation)
- No parallax effects
- Static backgrounds
- Simple fades only

## Visual Issues & Improvements

### Current Issues ⚠️

1. **ImportView Misleading**
   - Looks like import but manages bookmarks
   - Needs visual redesign to match function

2. **Mixed Paradigms**
   - File browser aesthetic
   - Import-style buttons
   - Confusing visual language

3. **Performance**
   - Poster generation blocks UI
   - No loading indicators
   - Synchronous operations visible

### Proposed Visual Improvements

1. **Rename & Redesign ImportView**
```yaml
new_design:
  title: "External Folders"
  icon: folder.badge.plus
  style: list of bookmarks
  action: "Add Folder" not "Import"
```

2. **Loading States**
```yaml
directory_loading:
  style: progress spinner
  position: inline with folder
  size: 14pt

poster_loading:
  style: skeleton
  shimmer: true
  duration: 1s
```

3. **Unified Visual Language**
- Remove import iconography
- Use folder/file metaphors consistently
- Clear browse vs play states

## Design Principles

1. **Glass Transparency**: Subtle materials that don't distract
2. **Folder Metaphor**: Filesystem navigation is primary
3. **DVD Library**: 2:3 posters maintain physical media feel
4. **Platform Native**: Respect each OS's conventions
5. **Progressive Disclosure**: Details on interaction
6. **Content Focus**: UI recedes during playback
7. **Accessibility First**: Clear focus, high contrast options

## Visual Testing Checklist

- [ ] Glass materials render correctly
- [ ] Focus states visible on all platforms
- [ ] Hover states work on macOS
- [ ] tvOS focus scaling smooth
- [ ] Dark/light mode transitions
- [ ] Poster loading doesn't block
- [ ] Animations respect reduce motion
- [ ] Text remains legible on glass
- [ ] Icons have sufficient contrast