---
id: "SPEC-UI-000"
title: "UI Design System Specification — Visual Language & Component Architecture"
owner: "@frontend-engineer"
status: "draft"
blueprint-ref: "13-user-experience/DESIGN_LANGUAGE.md"
version: "1.0.0"
---

# UI Design System Specification
## The Visual Language of Vestara — Across Workspace, OS, and Mobile

> **This document defines the complete visual design system for Vestara. It ensures that the Vestara AI OS, Workspace, mobile app, and future surfaces all feel like one cohesive product. This is a specification — not a CSS file. Implementation happens in code.**

---

## 1. 🎨 Color System

### Brand Colors
```yaml
name: "Vestara Brand Colors"

vestara:
  bg:
    value: "#06060C"
    usage: "Primary background (dark mode default)"
    token: "vest-bg"
  
  bg-elevated:
    value: "#0D0D14"
    usage: "Elevated surfaces (cards, modals, sidebar)"
    token: "vest-bg-elevated"

  bg-hover:
    value: "#14141E"
    usage: "Hover state for elevated surfaces"
    token: "vest-bg-hover"

  text:
    value: "#E8ECF1"
    usage: "Primary text color"
    token: "vest-text"

  text-muted:
    value: "#8B939E"
    usage: "Secondary text, captions, labels"
    token: "vest-text-muted"

  gold:
    value: "#C9A84C"
    usage: "Primary accent — brand highlight"
    token: "vest-gold"

  gold-hover:
    value: "#D4B85C"
    usage: "Gold hover states"
    token: "vest-gold-hover"

  border:
    value: "#1E1E28"
    usage: "Default border color"
    token: "vest-border"

  border-hover:
    value: "#2A2A38"
    usage: "Border hover states"
    token: "vest-border-hover"

  accent:
    value: "#3A3A4A"
    usage: "Secondary accent, focus rings"
    token: "vest-accent"
```

### Semantic Colors
```yaml
semantic:
  success:
    base: "#22C55E"
    bg: "#052E16"
    token: "vest-success"

  warning:
    base: "#F59E0B"
    bg: "#451A03"
    token: "vest-warning"

  error:
    base: "#EF4444"
    bg: "#450A0A"
    token: "vest-error"

  info:
    base: "#3B82F6"
    bg: "#0F172A"
    token: "vest-info"
```

### Light Mode (Future)
```yaml
light-mode:
  approach: "Invert dark base, preserve gold accent"
  status: "Gen 2"
```

---

## 2. 🔤 Typography

### Typeface
```yaml
family:
  primary:
    value: "'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif"
    usage: "UI text, headings, body"
  
  mono:
    value: "'JetBrains Mono', 'Fira Code', 'Cascadia Code', monospace"
    usage: "Code blocks, terminal, technical data"

  arabic:
    value: "'Noto Naskh Arabic', 'Amiri', serif"
    usage: "Arabic script support (Gen 3)"
```

### Type Scale
```yaml
scale:
  xs:
    size: "0.75rem"     # 12px
    line-height: 1rem   # 16px
    usage: "Captions, metadata, timestamps"

  sm:
    size: "0.875rem"    # 14px
    line-height: 1.25rem # 20px
    usage: "Body small, secondary text"

  base:
    size: "1rem"        # 16px
    line-height: 1.5rem # 24px
    usage: "Body text, paragraphs, buttons"

  lg:
    size: "1.125rem"    # 18px
    line-height: 1.75rem # 28px
    usage: "Large body, lead paragraphs"

  xl:
    size: "1.25rem"     # 20px
    line-height: 1.75rem # 28px
    usage: "H4 headings, section titles"

  2xl:
    size: "1.5rem"      # 24px
    line-height: 2rem   # 32px
    usage: "H3 headings, panel headers"

  3xl:
    size: "1.875rem"    # 30px
    line-height: 2.25rem # 36px
    usage: "H2 headings, page titles"

  4xl:
    size: "2.25rem"     # 36px
    line-height: 2.5rem # 40px
    usage: "H1 headings, welcome screen"

  5xl:
    size: "3rem"        # 48px
    line-height: 1
    usage: "Hero text, splash screen"

  6xl:
    size: "3.75rem"     # 60px
    line-height: 1
    usage: "Display text, marketing pages"
```

### Font Weights
```yaml
weights:
  normal: 400
  medium: 500
  semibold: 600
  bold: 700
```

---

## 3. 🎬 Motion Design

### Duration Principles
```yaml
principles:
  - "Motion should feel responsive, not slow"
  - "UI feedback: <100ms — instant"
  - "Transitions: 150-300ms — natural"
  - "Emphasis: 300-500ms — attention-drawing"
  - "Background animations: 500-1000ms — ambient"

durations:
  instant: "50ms"
  fast: "100ms"
  normal: "200ms"
  slow: "300ms"
  emphasize: "500ms"
  ambient: "1000ms"

easing:
  default: "cubic-bezier(0.4, 0, 0.2, 1)"    # Standard
  exit: "cubic-bezier(0.4, 0, 1, 1)"          # Accelerate
  enter: "cubic-bezier(0, 0, 0.2, 1)"         # Decelerate
  spring: "cubic-bezier(0.34, 1.56, 0.64, 1)" # Spring-like
```

### Animation Patterns

| Pattern | Duration | Easing | Usage |
|---------|----------|--------|-------|
| Fade in | 200ms | enter | Modal overlays, tooltips |
| Fade out | 150ms | exit | Dismissing overlays |
| Slide up | 300ms | spring | Panel opening |
| Slide down | 250ms | default | Dropdown, menu |
| Scale | 200ms | spring | Button press, card hover |
| Rotate | 200ms | default | Expand/collapse arrows |
| Progress | ambient | linear | Loading indicators |
| Skeleton pulse | 1500ms | linear | Content loading placeholder |

---

## 4. 📦 Spacing & Layout

```yaml
spacing:
  0: "0px"
  0.5: "0.125rem"   # 2px
  1: "0.25rem"       # 4px
  2: "0.5rem"        # 8px
  3: "0.75rem"       # 12px
  4: "1rem"          # 16px
  5: "1.25rem"       # 20px
  6: "1.5rem"        # 24px
  8: "2rem"          # 32px
  10: "2.5rem"       # 40px
  12: "3rem"         # 48px
  16: "4rem"         # 64px
  20: "5rem"         # 80px
  24: "6rem"         # 96px

breakpoints:
  sm: "640px"        # Mobile landscape
  md: "768px"        # Tablet
  lg: "1024px"       # Small desktop
  xl: "1280px"       # Large desktop
  2xl: "1536px"      # Ultra-wide

layout:
  max-width: "1440px"
  sidebar-width: "280px"
  panel-width: "360px"
  chat-width: "720px"
  input-min-height: "48px"
```

---

## 5. 🪄 Shadows & Elevation

```yaml
shadows:
  sm: "0 1px 2px 0 rgba(0,0,0,0.05)"
  base: "0 1px 3px 0 rgba(0,0,0,0.1), 0 1px 2px -1px rgba(0,0,0,0.1)"
  md: "0 4px 6px -1px rgba(0,0,0,0.1), 0 2px 4px -2px rgba(0,0,0,0.1)"
  lg: "0 10px 15px -3px rgba(0,0,0,0.1), 0 4px 6px -4px rgba(0,0,0,0.1)"
  xl: "0 20px 25px -5px rgba(0,0,0,0.1), 0 8px 10px -6px rgba(0,0,0,0.1)"
  glow-gold: "0 0 15px rgba(201, 168, 76, 0.3)"  # Brand accent glow

elevation:
  0: "flat"           # Base surface
  1: "shadow-sm"      # Cards, items
  2: "shadow-base"    # Dropdowns, tooltips
  3: "shadow-md"      # Modals, popovers
  4: "shadow-lg"      # Dialogs, alerts
  5: "shadow-xl"      # Full-screen overlays
```

---

## 6. 🔲 Border & Radius

```yaml
border:
  width:
    default: "1px"
    focused: "2px"
  
  radius:
    none: "0px"
    sm: "0.25rem"      # 4px
    md: "0.5rem"       # 8px
    lg: "0.75rem"      # 12px
    xl: "1rem"         # 16px
    2xl: "1.5rem"      # 24px
    full: "9999px"     # Circular

  style:
    default: "solid"
    dashed: "1px dashed $border"  # Drag zones, incomplete state
```

---

## 7. 🧩 Component Tokens

### Button

```yaml
component: Button

variants:
  primary:
    bg: "vest-gold"
    text: "vest-bg"
    hover-bg: "vest-gold-hover"
    height: "40px"
    padding: "12px 24px"
    radius: "0.5rem"
    font-weight: 600
  
  secondary:
    bg: "transparent"
    text: "vest-text"
    border: "vest-border"
    hover-bg: "vest-bg-hover"
    height: "40px"
    padding: "12px 24px"
    radius: "0.5rem"

  ghost:
    bg: "transparent"
    text: "vest-text-muted"
    hover-bg: "vest-bg-hover"
    height: "36px"
    padding: "8px 16px"
    radius: "0.375rem"

  danger:
    bg: "vest-error-bg"
    text: "vest-error"
    border: "vest-error"
    height: "40px"
    padding: "12px 24px"
    radius: "0.5rem"

states:
  disabled:
    opacity: 0.5
    cursor: "not-allowed"
  
  loading:
    show-spinner: true
    text-hidden: false
```

### Input

```yaml
component: Input

default:
  bg: "vest-bg"
  text: "vest-text"
  border: "vest-border"
  placeholder: "vest-text-muted"
  height: "40px"
  padding: "8px 12px"
  radius: "0.5rem"

focus:
  border: "vest-gold"
  ring: "0 0 0 2px rgba(201, 168, 76, 0.2)"

error:
  border: "vest-error"
  ring: "0 0 0 2px rgba(239, 68, 68, 0.2)"
```

### Card

```yaml
component: Card

default:
  bg: "vest-bg-elevated"
  border: "vest-border"
  radius: "0.75rem"
  shadow: "shadow-sm"
  padding: "24px"

interactive:
  hover-border: "vest-border-hover"
  hover-shadow: "shadow-md"
  cursor: "pointer"
```

### Modal

```yaml
component: Modal

overlay:
  bg: "rgba(0, 0, 0, 0.6)"
  backdrop-filter: "blur(4px)"

dialog:
  bg: "vest-bg-elevated"
  border: "vest-border"
  radius: "1rem"
  shadow: "shadow-lg"
  padding: "24px"
  max-width: "540px"
  animation: "scale(0.95) → scale(1), 200ms spring"
```

---

## 8. ♿ Accessibility

```yaml
accessibility:
  targets:
    wcag: "WCAG 2.1 AA (minimum), AAA (target)"

  contrast:
    normal-text: "4.5:1 minimum"
    large-text: "3:1 minimum"
    ui-components: "3:1 minimum"

  focus:
    style: "2px solid $vest-gold with 2px offset"
    visible: "always visible on keyboard navigation"

  keyboard:
    tab-order: "logical reading order"
    shortcuts: "all actions accessible via keyboard"
    command-palette: "Ctrl+P or Cmd+P"

  motion:
    prefers-reduced-motion: "respect OS setting"
    replace-animation: "cross-fade instead of slide/scale"

  screen-reader:
    labels: "all interactive elements have aria-labels"
    live-regions: "dynamic content uses aria-live"
    announcements: "notifications use role='alert'"
```

---

## 9. 🤖 AI Interaction Patterns

```yaml
ai-interaction-patterns:
  streaming:
    visual: "Typewriter effect with smooth character reveal"
    cursor: "Pulsing cursor during generation"
    speed: "Matching provider token rate"

  thinking:
    visual: "Subtle pulse on avatar/indicator"
    text: "Showing 'Thinking...' with duration"
    cancel: "Escape key or stop button always available"

  tool-executing:
    visual: "Inline progress bar within conversation"
    text: "Tool name and status shown"
    result: "Collapsible detail view for tool output"

  error:
    user: "Friendly error message with retry suggestion"
    technical: "Expandable technical details"
    recovery: "Auto-retry for transient errors (3 attempts)"

  offline:
    indicator: "Status badge in header"
    capability: "Graceful degradation — local models only"
    queue: "Messages queued for delivery when online"
```

---

## 10. 📱 Responsive Behavior

```yaml
responsive:
  desktop:
    breakpoint: ">= 1024px"
    layout: "Multi-panel — sidebar + main + optional sidecar"
    chat: "Full width within main panel"

  tablet:
    breakpoint: "768px - 1023px"
    layout: "Collapsible sidebar, single main panel"
    chat: "Full width"

  mobile:
    breakpoint: "< 768px"
    layout: "Single column, bottom navigation"
    chat: "Full screen, keyboard-aware"
    sidebar: "Overlay drawer from left"
```

---

## 11. 📝 Icons

```yaml
icons:
  library: "Lucide Icons"
  style: "outline, 1.5px stroke"
  sizes:
    sm: "16px"
    md: "20px"
    lg: "24px"
    xl: "32px"
  customization: "Stroke width adjustable, color inherits from text"
```

---

## 12. 📐 Design Token Architecture

```yaml
token-delivery:
  platform: "CSS custom properties (--vest-*)"
  tailwind: "tailwind.config.ts — extend theme with vestara-* tokens"
  figma: "Design tokens synced via Token Studio"
  versioning: "Design tokens versioned alongside code"

token-categories:
  - "Color (--vest-bg, --vest-text, --vest-gold)"
  - "Typography (--vest-font-sans, --vest-font-mono)"
  - "Spacing (--vest-space-*)"
  - "Shadows (--vest-shadow-*)"
  - "Radii (--vest-radius-*)"
  - "Animation (--vest-duration-*, --vest-ease-*)"
```

---

*This design system specification ensures that every Vestara surface — from Workspace to OS to mobile — shares the same visual DNA. Implementation follows these tokens, not the other way around.*
