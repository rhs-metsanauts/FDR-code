# UI Redesign Design Spec

**Date:** 2026-04-07  
**Status:** Approved  
**Source spec:** `UI_DESIGN - Copy.md`

---

## Context

The rover control app (`templates/index.html`) currently uses the METSAnauts website design language (Space Grotesk, `#070c16` background, `#5b9cf6` accent). This redesign replaces it with the mission-control dark theme from the UI design specification: Roboto fonts, `#212121` background, status-color system, and a dashboard split layout with a persistent right device panel.

All JavaScript behavior, HTML element IDs, and Flask endpoints remain unchanged. Only CSS and HTML structure change.

---

## Layout

### Overall structure

```
┌──────────────────────────────────────────────────────┐
│  TOP BAR (56px)                                       │
├────────────────────────────────┬─────────────────────┤
│                                │                     │
│  MAIN VIEWPORT (flex: 1)       │  RIGHT PANEL        │
│                                │  (340px fixed)      │
│  [Commands] [Map] tab bar      │                     │
│                                │  Device cards       │
│  Tab content:                  │  + Config section   │
│  - Commands form               │                     │
│  - 3D Map (Three.js)           │                     │
│                                │                     │
├────────────────────────────────┴─────────────────────┤
│  TELEMETRY BAR (48px)                                 │
└──────────────────────────────────────────────────────┘
```

### Top bar (56px)

- Left: 🚀 logo + "HERA ROVER COMMAND CENTER" (bold, uppercase, letter-spacing 2px) with "METSANAUTS MISSION CONTROL" subtitle in muted text below
- Right: system health badge driven by the last known state of both devices:
  - `● SYSTEMS NOMINAL` (green) — rover last command succeeded AND Jetson mapper connected
  - `● PARTIAL` (yellow) — one device is reachable but not both
  - `● NO SIGNAL` (red) — no confirmed connection to either device
  - Updated whenever a command result arrives or `/map_status` is polled

### Main viewport (left, `flex: 1`)

Contains the existing tab bar (Commands | Map) and all tab content. Background `#1a1a1a`.

Tab bar: `#212121` background, `border-bottom: 1px solid #333`. Active tab has `color: #2196F3` and `border-bottom: 2px solid #2196F3`. Tab labels uppercase, `font-size: 13px`, `font-weight: 500`, `letter-spacing: 0.5px`.

### Right panel (340px fixed width)

`background: #212121`, `border-left: 1px solid #333`. Two sections:

**Devices section** (top, `padding: 16px`):
- Section title: `DEVICES` in monospace, 10px, uppercase, letter-spacing 2px, color `#666`
- Rover card (Raspberry Pi) — purple border `#9C27B0`
- Mapper card (Jetson Nano) — cyan border `#00BCD4`; red `#F44336` border when offline

**Configuration section** (bottom, fills remaining height):
- Section title: `CONFIGURATION`
- All existing config fields: Rover Server URL, Jetson WS URL, Timeout, LoRa Destination
- Save Config button (outline style)

### Telemetry bar (48px, bottom)

Fixed bottom strip, `background: #212121`, `border-top: 1px solid #333`. Four items separated by `border-right: 1px solid #333`:

| Item | Data source | Colors |
|---|---|---|
| ROVER | Last command result (updated on send) | Green = last command succeeded, Red = last command failed, Gray = no command sent yet |
| MAPPER | `_jetson_ws_connected` via `/map_status` | Green = connected, Red = offline |
| MODE | Current WiFi/LoRa mode | Blue badge |
| POINTS | `_map_point_count` via `/map_status` | Cyan value |

---

## Color System

| Token | Hex | Usage |
|---|---|---|
| `--bg` | `#212121` | Page background |
| `--bg-deep` | `#1a1a1a` | Main viewport background |
| `--surface` | `#303030` | Cards, inputs |
| `--surface-2` | `#252525` | Disconnected card bg, config inputs |
| `--border` | `#333333` | Layout borders |
| `--border-soft` | `#444444` | Input borders, dividers |
| `--connected` | `#4CAF50` | Online/connected indicators |
| `--warning` | `#FF9800` | Low battery, weak signal |
| `--error` | `#F44336` | Disconnected, error state |
| `--active` | `#2196F3` | Selected/active, buttons, tab highlight |
| `--rpi` | `#9C27B0` | Raspberry Pi archetype color |
| `--jetson` | `#00BCD4` | Jetson archetype color |
| `--text` | `#FFFFFF` | Primary text |
| `--text-muted` | `#9E9E9E` | Secondary text, labels |
| `--text-dim` | `#666666` | Section titles, key labels |

---

## Typography

```css
/* Google Fonts import */
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&family=Roboto+Mono:wght@400;500&display=swap');
```

| Element | Font | Size | Weight |
|---|---|---|---|
| Body | Roboto | 14px | 400 |
| Top bar title | Roboto | 13px | 700 |
| Section headers | Roboto Mono | 10px | 400 |
| Tab labels | Roboto | 13px | 500 |
| Device card name | Roboto | 13px | 500 |
| Config labels | Roboto Mono | 11px | 400 |
| Input fields | Roboto Mono | 12–13px | 400 |
| Telemetry values | Roboto Mono | 12px | 400 |
| Command type chips | Roboto Mono | 12px | 400 |
| Send button | Roboto | 14px | 500 |

---

## Component Specs

### Device card

```
background: #303030
border-radius: 6px
padding: 14px
border: 2px solid <archetype-color>
```

- **Header row**: status dot (8px circle) + device name + archetype badge (small pill, semi-transparent archetype color bg)
- **Data rows**: key (`color: #666`) + value (`font-family: Roboto Mono`) in `justify-content: space-between` flex row
- **Disconnected state**: `border-color: #F44336`, `opacity: 0.7`, `background: #252525`
- **Connected glow** (when selected/active): `box-shadow: 0 0 16px rgba(<archetype>, 0.3)`

Rover card data rows: Connection, Mode, Address  
Mapper card data rows: Connection, Points, WebSocket URL

### Status dots

| State | Color | Glow |
|---|---|---|
| Connected | `#4CAF50` | `box-shadow: 0 0 6px #4CAF50` |
| Disconnected | `#F44336` | `box-shadow: 0 0 6px #F44336` |
| Warning | `#FF9800` | `box-shadow: 0 0 6px #FF9800` |

### Mode toggle (WiFi / LoRa)

```
background: #303030
border: 1px solid #444
border-radius: 4px
overflow: hidden
```

Active button: `background: #2196F3`, `color: #fff`. Inactive: `color: #9E9E9E`.

### Command type chips

Pill buttons. Inactive: `border: 1px solid #444`, `color: #9E9E9E`, `background: transparent`. Active: `border-color: #2196F3`, `color: #2196F3`, `background: rgba(33,150,243,0.1)`.

### Primary button (Send Command)

```
background: #2196F3
color: #fff
border-radius: 4px
padding: 13px
font-size: 14px
font-weight: 500
text-transform: uppercase
letter-spacing: 0.5px
width: 100%
```

Hover: `background: #1976D2`. Active: `transform: scale(0.99)`. Disabled: `opacity: 0.4`.

### Config inputs

```
background: #252525
border: 1px solid #3a3a3a
border-radius: 4px
color: #9E9E9E
font-family: Roboto Mono
font-size: 11px
padding: 7px 10px
```

Focus: `border-color: #2196F3`, `outline: none`.

### AI Assist FAB

Position: `fixed`, `bottom: 72px` (above telemetry bar), `right: 360px` (above right panel edge).  
`background: #2196F3`, `border-radius: 50%`, `width: 52px`, `height: 52px`.  
`box-shadow: 0 4px 20px rgba(33,150,243,0.5)`.  
Hover: `transform: scale(1.08)`.

### Response section

Success: `border-left: 3px solid #4CAF50`  
Error: `border-left: 3px solid #F44336`  
Background: `#303030`, `border-radius: 4px`

---

## Structural HTML Changes

### 1. Remove old CSS variables and fonts

Remove the existing `:root { --bg: #070c16; ... }` block and the Space Grotesk / JetBrains Mono Google Fonts link. Replace with Roboto import and new CSS variables.

### 2. Restructure `<body>`

Current structure:
```
<div class="container">
  <div class="header">...</div>
  <div class="tab-bar">...</div>
  <div class="content" id="tab-commands">...</div>
  <div id="tab-map">...</div>
</div>
```

New structure:
```
<div class="top-bar">...</div>
<div class="main-area">
  <div class="main-viewport">
    <div class="tab-bar">...</div>
    <div class="viewport-content" id="tab-commands">...</div>
    <div class="viewport-content hidden" id="tab-map">...</div>
  </div>
  <div class="right-panel">
    <div class="panel-section" id="devices-section">
      <!-- Rover card -->
      <!-- Jetson card -->
    </div>
    <div class="panel-section" id="config-section">
      <!-- Config fields (moved from tab-commands) -->
    </div>
  </div>
</div>
<div class="telemetry-bar">...</div>
```

The `.header` div (NASA logo + title) is removed — replaced by `.top-bar`.  
The config section currently inside `#tab-commands` moves to `#config-section` in the right panel.

### 3. Element IDs preserved

All existing IDs stay exactly as-is: `serverUrl`, `timeout`, `loraDestination`, `commandType`, `actionField`, `commandField`, `fileNameField`, `fileContentField`, `readFileNameField`, `readImageNameField`, `responseSection`, `modeWifi`, `modeLora`, `tabCmdBtn`, `tabMapBtn`, `tab-commands`, `tab-map`, `mapCanvas`, `minimapCanvas`, `mapStatusDot`, `mapStatusText`, `mapPointCount`, `mapCoverage`, `mapElapsed`.

### 4. New elements added

- `#roverStatusDot` — status dot in Rover device card
- `#roverStatusText` — "Online" / "Offline" text in Rover card  
- `#roverModeText` — current mode value in Rover card
- `#roverAddressText` — current server address in Rover card
- `#jetsonStatusDot` — status dot in Jetson device card
- `#jetsonStatusText` — "Connected" / "Offline" in Jetson card
- `#jetsonPointsText` — point count in Jetson card
- `#telemRoverDot`, `#telemRoverText` — telemetry bar rover status
- `#telemMapperDot`, `#telemMapperText` — telemetry bar mapper status
- `#telemModeBadge` — mode badge in telemetry bar
- `#telemPoints` — point count in telemetry bar

---

## JavaScript Changes

### Update `_pollMapStatus()`

Currently updates `#mapStatusDot` and `#mapStatusText`. Extend to also update:
- `#jetsonStatusDot` / `#jetsonStatusText` / `#jetsonPointsText` in the Jetson device card
- `#telemMapperDot` / `#telemMapperText` / `#telemPoints` in the telemetry bar

### Update mode switching

When WiFi/LoRa mode toggles, update `#roverModeText` in the Rover card and `#telemModeBadge` in the telemetry bar.

### Update `switchTab()`

Currently toggles `#tab-commands` and `#tab-map`. No changes needed — IDs preserved.

### Top bar system health

Add `#topBarDot` and `#topBarStatusText` to the top bar. Update them whenever:
- A command is sent (success → rover is reachable; error → rover unreachable)
- `/map_status` is polled (provides `_jetson_ws_connected`)

Logic: if last rover command succeeded AND `connected === true` → green "SYSTEMS NOMINAL"; if exactly one → yellow "PARTIAL"; if neither → red "NO SIGNAL". Initial state on page load: gray "STANDBY".

---

## What Does NOT Change

- All JavaScript functions and logic
- All `onclick` handlers and event listeners
- The Three.js scene, SSE consumer, minimap, elapsed timer
- The AI modal HTML and streaming logic
- All Flask endpoints and backend code
- The `.hidden` utility class behavior
