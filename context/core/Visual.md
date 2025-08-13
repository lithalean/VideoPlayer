# VideoPlayer Visual Design Context

**Purpose**: Complete visual design system for QuickTime-style video player  
**Version**: 1.0  
**Design Language**: Apple HIG + DVD Library Aesthetic  
**Last Updated**: August 12, 2025

## Visual Hierarchy
```
Z-LAYERS (back→front):
├─[0] Background (black/gradient)
├─[1] Content (grids/posters)
├─[2] Controls (buttons/overlays)
└─[3] Overlays (sheets/fullscreen)
```

## Quick Reference Tables

### Color Palette
| Element | Light Mode | Dark Mode | Usage |
|---------|------------|-----------|-------|
| **Background** | .systemBackground | .black | Base layer |
| **Poster Gradient Start** | .accentColor(0.55) | Same | Fallback poster top |
| **Poster Gradient Mid** | .accentColor(0.25) | Same | Fallback poster middle |
| **Poster Gradient End** | .black(0.25) | Same | Fallback poster bottom |
| **Bottom Fade** | .black(0.28) | Same | Poster overlay gradient |
| **Glass Material** | .regularMaterial | .ultraThinMaterial | Controls/bars |
| **Shadow iOS** | .black(0.12) | .black(0.12) | Poster depth |
| **Shadow tvOS** | .black(0.30) | .black(0.30) | Enhanced depth |
| **Duration Pill** | .black(0.55) | Same | Time overlay background |
| **Backdrop** | .accentColor(0.45→0.18) | Same | MovieView gradient |

### Spacing System
| Token | iOS/macOS | tvOS | Usage |
|-------|-----------|------|-------|
| **xs** | 2pt | 4pt | Micro adjustments |
| **small** | 8pt | 12pt | Internal padding |
| **medium** | 16pt | 24pt | Component spacing |
| **large** | 20pt | 32pt | Section gaps |
| **grid** | 16pt | 24pt | Poster grid spacing |
| **sidebar** | 12pt | 16pt | Sidebar padding |

### Typography Scale
| Style | iOS/macOS | tvOS | Weight | Usage |
|-------|-----------|------|--------|-------|
| **Overlay Title** | .title | .largeTitle | Bold | Movie title on playback |
| **Grid Title** | .subheadline | .headline | Semibold | Poster card labels |
| **Duration** | .caption2 | .caption2 | Regular | Duration pills |
| **Badge** | .caption2 | .caption2 | Regular | Sidebar counts |
| **Button** | .headline | .title3 | Medium | Play button |
| **Volume** | .caption | .caption | Regular | Volume percentage |

### Corner Radius
| Component | iOS/macOS | tvOS | Style |
|-----------|-----------|------|-------|
| **Poster** | 12pt | 18pt | continuous |
| **Glass Bar** | 28pt | 28pt | continuous |
| **Buttons** | 10pt | 12pt | continuous |
| **Pills** | full | full | capsule |

## Component Specifications

### MoviePoster (2:3 DVD Aspect)
```yaml
aspect_ratio: 2/3  # DVD/Blu-ray standard
corner_radius:
  ios: 12pt
  tvos: 18pt
  style: continuous
shadow:
  ios: 
    color: black(0.12)
    radius: 8pt
    x: 0
    y: 4pt
  tvos:
    color: black(0.30)
    radius: 18pt
    x: 0
    y: 8pt
bottom_fade:
  gradient_stops:
    - { color: clear, location: 0.60 }
    - { color: black(0.17), location: 0.82 }
    - { color: black(0.28), location: 1.00 }
  direction: top_to_bottom
fallback:
  gradient:
    colors: [accentColor(0.55), accentColor(0.25), black(0.25)]
    direction: topLeading_to_bottomTrailing
  icon:
    symbol: "film"
    size: { ios: 42pt, tvos: 60pt }
    weight: light
    opacity: 0.9
```

### Movies Grid Layout
```yaml
type: LazyVGrid
columns:
  ios: 
    adaptive: { min: 140pt, max: 220pt }
    spacing: 16pt
    alignment: top
  tvOS:
    adaptive: { min: 260pt, max: 360pt }
    spacing: 24pt
    alignment: top
grid_spacing:
  ios: 16pt
  tvOS: 24pt
card_structure:
  - poster: MoviePoster component
  - duration_pill:
      position: bottomTrailing
      inset: 8pt
      background: black(0.55)
      text: white
      padding: { h: 6pt, v: 3pt }
      shape: capsule
  - title:
      font: { ios: subheadline, tvos: headline }
      weight: semibold
      lines: 2
      alignment: leading
focus_effect:
  tvOS:
    scale: 1.08
    animation: easeInOut(0.2s)
```

### Shows Grid (Variable Sizes)
```yaml
type: LazyVGrid
columns:
  ios: 6 flexible, spacing: 16pt
  tvOS: 4 flexible, spacing: 24pt
card_sizes:
  pattern: [large, medium, medium, medium, small, small, small, medium, medium, medium, medium, large]
  small:
    height: { ios: 160pt, tvos: 280pt }
    column_span: 1
  medium:
    height: { ios: 200pt, tvos: 320pt }
    column_span: 2
  large:
    height: { ios: 240pt, tvos: 380pt }
    column_span: 3
expansion_effect:
  scale: 1.02
  animation: spring(0.35, 0.8)
```

### MovieView Playback Screen
```yaml
backdrop:
  type: LinearGradient
  stops:
    - { color: accentColor(0.45), location: 0.0 }
    - { color: accentColor(0.18), location: 0.55 }
    - { color: black(0.22), location: 1.0 }
  direction: topLeading_to_bottomTrailing
  fade_animation:
    trigger: play_button
    duration: 250ms
    curve: easeInOut
    opacity: 1.0 → 0.0
overlay_controls:
  title:
    font: { ios: title, tvos: largeTitle }
    weight: bold
    shadow: { radius: 4pt }
  play_button:
    background: white
    foreground: black
    padding: { h: 18pt, v: 10pt }
    shape: capsule
    icon_offset: { playing: 0, paused: 2pt }
  back_button:
    icon: "chevron.left"
    background: ultraThinMaterial
    shape: circle
    padding: 10pt
  fullscreen_button:
    icon: "arrow.up.left.and.arrow.down.right"
    background: ultraThinMaterial
    shape: capsule
    padding: { h: 12pt, v: 8pt }
```

### Glass Playback Bar (WWDC 2025 Style)
```yaml
container:
  background:
    base: Color(white: 0.1, opacity: 0.8)
    overlay: ultraThinMaterial(0.3)
    corner_radius: 28pt
  border:
    gradient: [white(0.2), white(0.05)]
    width: 0.5pt
  shadow:
    color: black(0.4)
    radius: 20pt
    x: 0
    y: 10pt
controls:
  play_pause:
    background:
      shape: circle
      fill: white(0.12)
      size: 56pt
    icon:
      size: 26pt
      weight: medium
      offset: { play: 2pt, pause: 0 }
  seek:
    icon: "goforward/backward.10"
    size: 26pt
    spacing: 28pt
  airplay:
    icon: "airplayvideo"
    size: 22pt
    opacity: 0.9
volume:
  ios_macos:
    track:
      height: 4pt
      fill: white(0.15)
    progress:
      height: 4pt
      fill: white(0.7)
    width: 120pt
    gesture: drag
  tvos:
    buttons: [minus, percentage, plus]
    spacing: 8pt
    icon_size: 14pt
```

## Platform Visual Adaptations

### iOS
```yaml
navigation: split_view_with_sidebar
poster_sizes: 140-220pt
shadows: subtle (0.12 opacity)
corner_radii: 12pt standard
fullscreen: sheet presentation
gestures: drag for volume
animations: standard iOS
```

### tvOS
```yaml
navigation: stack (no split)
poster_sizes: 260-360pt (1.5-1.8x iOS)
shadows: enhanced (0.30 opacity)
corner_radii: 18pt (1.5x iOS)
focus_effects:
  scale: 1.08-1.10
  animation: 200ms
  material_change: regular when focused
controls: button-based (no drag)
```

### macOS
```yaml
navigation: split_view
titlebar_spacing: 28pt (traffic lights)
window_chrome: native
fullscreen: window toggle
hover_states: enabled
cursor: pointer on interactive
shadows: same as iOS
```

## Animation Specifications

### Standard Timings
```yaml
instant: 0ms
fast: 100ms
normal: 200ms
smooth: 250ms
slow: 350ms
dramatic: 500ms
```

### Poster Focus (tvOS)
```yaml
trigger: @FocusState change
properties:
  scale: 1.0 → 1.08
  shadow: enhanced
duration: 200ms
curve: easeInOut
```

### Playback Overlay Fade
```yaml
trigger: isPlaying state change
targets:
  - backdrop gradient
  - overlay controls
  - title/buttons
opacity: 1.0 → 0.0
duration: 250ms
curve: easeInOut
```

### Grid Card Tap (Shows)
```yaml
trigger: tap gesture
scale: 1.0 → 1.02
duration: 200ms
curve: spring(response: 0.35, damping: 0.8)
```

### Volume Adjust
```yaml
trigger: drag or button
feedback: haptic (iOS)
animation: none (real-time)
```

## Visual Polish Details

### Poster Quality
- Format: JPEG
- Compression: 0.85
- Max dimensions: 1200x1800
- Frame selection: 10% into video
- Color space: sRGB
- Transform: Applied for orientation

### Glass Effects
```yaml
materials:
  ultra_thin: opacity(0.3), blur(20)
  regular: opacity(0.5), blur(30)
  thick: opacity(0.8), blur(40)
borders:
  gradient: top(light) to bottom(dark)
  width: 0.5pt
shadows:
  multiple: true
  layers: [ambient, key_light]
```

### Typography Polish
- Monospaced digits: Duration displays
- Line limits: 2 for titles
- Truncation: tail
- Alignment: leading for cards, center for overlays
- Kerning: default
- Dynamic Type: supported

## Accessibility

### Contrast Ratios
- Text on dark: 7:1 minimum
- Controls: 4.5:1 minimum
- Focus indicators: 3:1 minimum

### Motion
- Respects reduce motion
- Alternative: instant transitions
- Focus visible: always

## Design Principles

1. **DVD Library Aesthetic**: Physical media metaphor
2. **Progressive Disclosure**: Details on interaction
3. **Platform Native**: Respect each OS
4. **Depth Through Layers**: Multi-level shadows
5. **Purposeful Motion**: Guide attention
6. **Subtle Glass**: Modern without distraction
7. **Content First**: UI recedes during playback