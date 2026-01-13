# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Windhawk "tool mod"** that enhances the Windows Explorer "Up" button. It opens the parent folder in a new Explorer window when:
- Middle-click on the Up button
- Ctrl + Left-click on the Up button
- Ctrl + Alt + Up keyboard shortcut

## Build System

This is a Windhawk module - there is no separate build step. The `.wh.cpp` file is compiled directly by Windhawk when the mod is installed/enabled. Required linker flags are specified in the file header:

```
// @compilerOptions -lole32 -loleaut32 -luiautomationcore -lshell32 -luser32 -luuid
```

To test changes, edit the file in Windhawk's editor or reload the mod.

## Architecture

### Tool Mod Pattern

This mod uses the "Mods as tools" pattern (runs in dedicated `windhawk.exe` process, not injected into Explorer):

1. **Target**: `@include windhawk.exe`
2. **Launcher layer** (`Wh_ModInit`, `Wh_ModAfterInit`, `Wh_ModUninit`): Standard boilerplate that spawns a dedicated process with `-tool-mod "mod-id"`
3. **Tool layer** (`WhTool_ModInit`, `WhTool_ModUninit`): Actual mod logic

Benefits: Shell stability (crashes don't affect Explorer), works with multiple Explorer processes.

### Core Technologies

- **Low-Level Hooks**: `WH_MOUSE_LL` and `WH_KEYBOARD_LL` installed from dedicated thread with message pump
- **UI Automation (UIA)**: Detects "Up" button via `AutomationId == "UpButton"` or localized `Name` prefix
- **COM**: Thread-local initialization with `COINIT_MULTITHREADED`

### Action Sequence

When triggered: `Ctrl+N` (duplicate window) → wait for new window → `Alt+Up` (navigate to parent)

### Key Constants

- `kMouseDebounceMs = 200` - Ignore clicks too close together
- `kNewWindowWaitMs = 700` - Time to wait for new Explorer window
- `kMidCooldownMs = 1000` - Cooldown after activation

## Debugging

- Use `Wh_Log(L"format", ...)` for debug output (visible in Windhawk's log panel)
- The mod runs in a separate `windhawk.exe` process
- COM must be initialized on each thread that uses UIA (handled by `EnsureUiaForThisThread`)
