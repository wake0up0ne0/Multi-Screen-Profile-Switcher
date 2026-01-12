# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Multi-Screen Profile Switcher is a portable Windows tray application that allows users to save, remove, and switch between multiple screen profiles with a single click. Each profile includes resolution, position, orientation, and refresh rate settings.

## Build & Run

**Build the project:**
```
msbuild screenTray.sln /p:Configuration=Release
```

**Debug build:**
```
msbuild screenTray.sln /p:Configuration=Debug
```

**Output locations:**
- Release: `bin\Release\MultiScreenProfileSwitcher.exe`
- Debug: `bin\Debug\MultiScreenProfileSwitcher.exe`

**Platform-specific builds:**
```
msbuild screenTray.sln /p:Configuration=Release /p:Platform=x64
```

## Technical Requirements

- **Target Framework:** .NET Framework 4.8.1
- **Output Type:** WinExe (Windows Forms application)
- **Platform:** Windows only (uses Win32 APIs)
- **Dependencies:** Newtonsoft.Json 13.0.3

## Architecture

### Entry Point & Application Lifecycle
**Program.cs** - Contains `Main()` entry point with:
- Mutex to prevent multiple instances (uses app name "Multi-Screen Profile Switcher")
- Launches `TaskTrayApplicationContext` which handles the entire application lifecycle

### Core Components

**TaskTrayApplicationContext.cs** - Main application logic:
- Manages NotifyIcon (system tray icon) and context menu
- Handles profile persistence via JSON (config.json in executable directory)
- Stores profiles in `Dictionary<string, screenState[]>` structure
- "default" profile is always the screen state at application startup
- Registry integration for Windows startup (HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run)
- Custom InputBox implementation for profile naming

**setScreen.cs** - Low-level screen manipulation:
- P/Invoke wrappers for Win32 APIs: `EnumDisplaySettings`, `ChangeDisplaySettings`, `ChangeDisplaySettingsExA`
- `DEVMODE` struct definition for display settings
- `screenState` struct: serializes to format `name;x;y;width;height;rotate;hz`
- `getScreenState()` - Queries current display configuration from all screens
- `setScreenState()` - Applies saved configuration using Win32 APIs
- Note: Scale settings are NOT supported (Windows API limitation)

### Profile Data Flow
1. User saves profile → Current screen state captured via `SetScreen.getScreenState()`
2. Stored in dictionary with user-provided name
3. Serialized to JSON and written to config.json
4. Loading profile → Deserialize from dictionary, apply via `SetScreen.setScreenState()`

### Key Menu Actions
- **Save Current Profile** - Captures current screen state, prompts for name, validates against existing profiles
- **Load Profiles** - Dynamically generated menu showing all saved profiles (checked state indicates current)
- **Remove Profiles** - Delete saved profiles (cannot remove "default")
- **Auto Open at Startup** - Toggle registry entry for Windows startup
- **Double-click tray icon** - Loads "default" profile (resets to startup state)

## Important Implementation Details

- Profile comparison uses string serialization of `screenState[]` arrays
- Application uses `ApplicationContext` pattern instead of standard Form
- All screen APIs are synchronous and return error codes (0 = success)
- JSON config file is loaded at startup and merged with runtime "default" profile
- Menu items are dynamically regenerated via `renewLoad()` after any profile changes
