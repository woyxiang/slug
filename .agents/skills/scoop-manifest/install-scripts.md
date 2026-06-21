# Install Scripts & Persistent Data

> Adapted from [Scoop Wiki - Pre/Post Install Scripts](https://github.com/ScoopInstaller/Scoop/wiki/Pre-Post-(un)install-scripts) and [Persistent Data](https://github.com/ScoopInstaller/Scoop/wiki/Persistent-data)

Cross-references: [manifest-schema.md](manifest-schema.md) | [autoupdate.md](autoupdate.md) | [workflow.md](workflow.md)

## Script Variables

Available in `pre_install`, `post_install`, `pre_uninstall`, `post_uninstall`, `installer.script`, `uninstaller.script`:

| Variable | Example | Description |
|---|---|---|
| `$app` | `myapp` | Manifest filename (without .json) |
| `$architecture` | `64bit` | CPU architecture being installed |
| `$cmd` | `install` | Current subcommand: `install`, `update`, `uninstall` |
| `$version` | `1.2.3` | Version being installed |
| `$global` | `$false` | `$true` for global installs |
| `$manifest` | (object) | Deserialized manifest as PowerShell object |
| **Path variables** | | |
| `$dir` | `~\scoop\apps\myapp\current` | Install directory |
| `$persist_dir` | `~\scoop\persist\myapp` | Persistent data directory |
| `$bucketsdir` | `~\scoop\buckets` | Buckets directory |
| `$cachedir` | `~\scoop\cache` | Download cache |
| `$scoopdir` | `~\scoop` | Base Scoop directory |

> **Note:** `$dir` expands to the version path in pre/post scripts but to `current` in `post_install`.

## Referencing Other Apps

Use `appdir` to reference another Scoop app's install directory:

```json
"post_install": [
    "if (Test-Path \"$(appdir otherapp)\\current\\otherapp.exe\") { ... }"
]
```

## Persistent Data

Files/directories that survive updates, stored in `~/scoop/persist/<app>/`.

### Simple persist

```json
"persist": ["config", "data"]
```

### Persist with rename

```json
"persist": [
    "keeps_its_name",
    ["original_name", "renamed_in_persist_dir"]
]
```

### String shorthand (single item)

```json
"persist": "config"
```

## Common Script Patterns

### Copy config from user data on first install

```json
"pre_install": [
    "if (!(Test-Path \"$persist_dir\\config\")) {",
    "    Copy-Item -ErrorAction Ignore \"$env:APPDATA\\app\\config\" \"$dir\\config\"",
    "    New-Item -ErrorAction Ignore \"$dir\\config\" | Out-Null",
    "}"
]
```

### Set environment variable pointing to install dir

```json
"env_set": {
    "APP_HOME": "$dir"
}
```

### Add subdirectory to PATH

```json
"env_add_path": "bin"
```

### Post-install message

```json
"notes": [
    "Usage: Add 'Invoke-Expression (&tool init powershell)' to your $PROFILE."
]
```

### Registry import (context menu, file associations)

```json
"post_install": [
    "$content = Get-Content -Path \"$dir\\install-context.reg\" -Encoding utf8",
    "$content -replace '{{app_dir}}', ($dir -replace '\\\\', '\\\\') | Set-Content -Path \"$dir\\install-context.reg\" -Encoding unicode",
    "reg import \"$dir\\install-context.reg\""
]
```
