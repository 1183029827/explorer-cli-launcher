---
name: explorer-cli-launcher
description: |
  Use this skill whenever the user wants to launch a CLI application directly from the Windows File Explorer address bar. The user might say things like "I want to type X in the Explorer address bar to open it", "how do I launch X from Explorer", "make X work from the address bar", "let me open X directly instead of opening PowerShell first", or mention typing a CLI name in Explorer and having it work. This skill creates a .bat launcher script in a PATH directory so that typing the app name in Explorer's address bar automatically opens PowerShell and runs the CLI.
---

# Explorer CLI Launcher

Creates `.bat` launcher scripts so that typing a CLI app name in the Windows File Explorer address bar opens PowerShell and runs the app directly.

## Why this is needed

Typing a CLI name (like `claude`) in Explorer's address bar usually fails or gives a bad experience:
- `.cmd` files are found in PATH but open in a bare cmd window that flashes away
- Direct `.exe` CLI tools (TUI apps) may not work properly without a terminal host
- The user currently has to type `powershell`, wait, then type the CLI name — two steps

The fix: a `.bat` file in a PATH directory that uses `start powershell -NoExit -Command "appname.cmd %*"`. Explorer finds the `.bat` (higher priority than `.cmd` in PATHEXT), launches PowerShell properly, and forwards any arguments.

## Workflow — Part A: Address Bar Launcher

### Step A1: Identify the CLI app

Ask the user which CLI app(s) they want to launch. The name should be the command they'd type in PowerShell — e.g., `claude`, `opencode`, `node`, `npm`.

### Step A2: Find the best PATH directory

Run the following PowerShell command to list the user's PATH directories:

```powershell
$env:PATH -split ';' | Select-Object -First 20
```

Then pick the best directory, in order of preference:

1. `%USERPROFILE%\.local\bin` — if it exists and is in PATH (usually first entry), this is ideal
2. `%APPDATA%\npm` — where npm global packages live, already in PATH
3. Any other user-writable directory early in the PATH

Check if the chosen directory exists. If it doesn't, create it and warn the user they may need to re-login for PATH changes to take effect if the directory wasn't already in PATH.

### Step A3: Check for conflicts

Before creating, check if a `.bat` or `.cmd` file with the same name already exists in the target directory:

```powershell
Test-Path -LiteralPath "$env:USERPROFILE\.local\bin\APPNAME.bat"
Test-Path -LiteralPath "$env:USERPROFILE\.local\bin\APPNAME.cmd"
```

If a `.bat` already exists, ask the user if they want to overwrite it. If only a `.cmd` exists, that's fine — `.bat` takes precedence in PATHEXT and will be found first by Explorer.

### Step A4: Create the .bat file

Write the `.bat` file with this content:

```batch
@echo off
start powershell -NoExit -Command "APPNAME.cmd %*"
```

Replace `APPNAME` with the actual CLI name.

**Important details:**
- Use `APPNAME.cmd` (with explicit `.cmd` extension) inside the PowerShell command, not just `APPNAME`, to prevent PowerShell from recursively finding this `.bat` file
- `%*` forwards any arguments the user types after the command name in Explorer
- `start` opens a new window (Explorer has no terminal to attach to)
- `-NoExit` keeps the PowerShell window open after the CLI exits

If the CLI is a plain `.exe` (not `.cmd`), use the `.exe` name directly: `APPNAME.exe %*`.

### Step A5: Verify

Tell the user to test by typing the app name in File Explorer's address bar and pressing Enter. A PowerShell window should open with the CLI running.

---

## Workflow — Part B: Right-Click Context Menu

Adds right-click context menu entries so that the CLI app can be launched from any folder, with that folder as the working directory. Supports two entry types:

- **Folder context menu** (right-click on a folder) — uses that folder as workspace
- **Folder background menu** (right-click empty space in a folder) — uses the current folder as workspace

### Step B1: Decide which menu entries to create

Ask the user which entries they want. Common choices:

| Entry | Registry Path | Variable |
|-------|--------------|----------|
| Right-click a folder | `HKCU:\Software\Classes\Directory\shell\` | `%1` |
| Right-click folder background | `HKCU:\Software\Classes\Directory\Background\shell\` | `%V` |

Both can be created at the same time. If the user doesn't specify, default to both.

### Step B2: Create the registry entries

For each context menu location, create a registry key under `HKCU:\Software\Classes\<path>\shell\<SubKey>` with these values:

- **(Default)** — The display text in the context menu, e.g. `"Open with OpenCode"`
- **Icon** (optional) — Path to an icon file, or the CLI executable path to use its built-in icon
- **Extended** (optional) — If set to empty string, the menu item only appears when holding Shift

Under the `<SubKey>` key, create a `command` subkey with:
- **(Default)** — The command to execute

The command template:

```
powershell -NoExit -Command "Set-Location -LiteralPath '%V'; APPNAME.cmd"
```

Replace `%V` with `%1` for the folder context menu type.

**Full PowerShell commands to create both menu entries at once:**

```powershell
# Folder context menu (right-click on a folder)
$name = "Open with OpenCode"
$appCmd = "opencode.cmd"
$regPath = "HKCU:\Software\Classes\Directory\shell\opencode"
New-Item -Path $regPath -Force | Out-Null
Set-ItemProperty -Path $regPath -Name "(Default)" -Value $name
New-Item -Path "$regPath\command" -Force | Out-Null
Set-ItemProperty -Path "$regPath\command" -Name "(Default)" -Value "powershell -NoExit -Command `"Set-Location -LiteralPath '%1'; $appCmd`""

# Folder background context menu (right-click empty space)
$bgRegPath = "HKCU:\Software\Classes\Directory\Background\shell\opencode"
New-Item -Path $bgRegPath -Force | Out-Null
Set-ItemProperty -Path $bgRegPath -Name "(Default)" -Value $name
New-Item -Path "$bgRegPath\command" -Force | Out-Null
Set-ItemProperty -Path "$bgRegPath\command" -Name "(Default)" -Value "powershell -NoExit -Command `"Set-Location -LiteralPath '%V'; $appCmd`""
```

**Customize for any app** by changing `$name`, `$appCmd`, and the subkey name (`opencode`) to match the target CLI app.

### Step B3: Verify context menu

Ask the user to right-click a folder (or empty space in a folder) in File Explorer. The new menu entry should appear. Changes take effect immediately — no restart needed.

### Step B4: Remove context menu entries

To remove previously added entries:

```powershell
Remove-Item -LiteralPath "HKCU:\Software\Classes\Directory\shell\opencode" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item -LiteralPath "HKCU:\Software\Classes\Directory\Background\shell\opencode" -Recurse -Force -ErrorAction SilentlyContinue
```

## Multiple apps

If the user wants to set up multiple CLI apps, create a `.bat` for each one in the same PATH directory. Process them in parallel — each is independent. Context menu entries can also be added for each app independently.
