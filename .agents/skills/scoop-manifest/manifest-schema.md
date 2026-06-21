# App Manifest Schema

> Adapted from [Scoop Wiki - App Manifests](https://github.com/ScoopInstaller/Scoop/wiki/App-Manifests)

Cross-references: [autoupdate.md](autoupdate.md) | [install-scripts.md](install-scripts.md) | [examples.md](examples.md) | [workflow.md](workflow.md)

## Minimal Example

```json
{
    "version": "1.0",
    "url": "https://github.com/user/repo/archive/master.zip",
    "extract_dir": "repo-master",
    "bin": "program.exe"
}
```

## Required Properties

| Property | Type | Description |
|---|---|---|
| `version` | string | Version of the app this manifest installs |
| `description` | string | One-line description. Don't repeat the app name if it matches the filename |
| `homepage` | string | Project home page URL |
| `license` | string or object | SPDX identifier, or `{identifier, url}` object. Use "Freeware", "Proprietary", "Public Domain", "Shareware", or "Unknown" for non-SPDX. Comma-separate different-file licenses, pipe (`|`) for dual-licensed |

## Optional Properties

| Property | Type | Description |
|---|---|---|
| `architecture` | object | Per-arch config: `32bit`, `64bit`, `arm64`. Each can contain: `url`, `hash`, `bin`, `checkver`, `extract_dir`, `installer`, `pre_install`, `post_install`, `shortcuts`, `uninstaller` |
| `url` | string or array | Download URL(s). Append `#/dl.7z` fragment to rename download (e.g. force 7z extraction of .exe) |
| `hash` | string or array | SHA256 hash (default). Prefix with `sha512:`, `sha1:`, or `md5:` for other algorithms |
| `bin` | string, array, or alias | Executable(s) to add to PATH. See [Binary Formats](#binary-formats) below |
| `extract_dir` | string | Subdirectory inside archive to extract |
| `extract_to` | string | Extract all content to this directory (relative to install dir) |
| `checkver` | string or object | Version detection config. See [autoupdate.md](autoupdate.md) |
| `autoupdate` | object | Auto-update URL templates. See [autoupdate.md](autoupdate.md) |
| `depends` | array | Runtime dependencies (installed automatically) |
| `suggest` | object | Optional companion tools. Format: `{"Feature": ["app1", "app2"]}` |
| `env_set` | object | Environment variables to set. Values support `$dir`, `$persist_dir`, `$version` |
| `env_add_path` | string or array | Directories (relative to install dir) to add to PATH |
| `notes` | string or array | Post-install messages to display |
| `persist` | string or array | Files/dirs to persist across updates. See [install-scripts.md](install-scripts.md) |
| `pre_install` | string or array | PowerShell commands run before installation |
| `post_install` | string or array | PowerShell commands run after installation |
| `pre_uninstall` | string or array | PowerShell commands run before uninstallation |
| `post_uninstall` | string or array | PowerShell commands run after uninstallation |
| `installer` | object | Custom installer: `{file, script, args, keep}` |
| `uninstaller` | object | Custom uninstaller: `{file, script, args}` |
| `innosetup` | boolean | `true` if installer is InnoSetup-based |
| `shortcuts` | array | Start menu shortcuts: `[[exe, name], [exe, name, args, icon]]` |
| `psmodule` | object | Install as PowerShell module: `{name}` |
| `##` | string or array | Comments (not used by Scoop, for human readers) |

## Binary Formats

**Simple executable:**
```json
"bin": "program.exe"
```

**Multiple executables:**
```json
"bin": ["bin\\program.exe", "tool.exe"]
```

**Alias (different name on PATH):**
```json
"bin": [["program.exe", "alias-name"]]
```

**Alias with arguments:**
```json
"bin": [["program.exe", "alias", "--flag"]]
```

**Mixed:**
```json
"bin": [
    "tool1.exe",
    ["tool2.exe", "mytool"],
    "bin\\tool3.exe"
]
```

> **Important:** If you declare just one shim as an alias array, wrap it in an outer array:
> `"bin": [ [ "program.exe", "alias" ] ]`. Otherwise it's read as separate shims.

## License Object Format

**Simple SPDX string:**
```json
"license": "MIT"
```

**Multiple licenses (different files):**
```json
"license": "GPL-2.0-only, MIT"
```

**Dual-licensed (same file):**
```json
"license": "GPL-2.0-only|MIT"
```

**Object with URL:**
```json
"license": {
    "identifier": "BSD-2-Clause, BSD-3-Clause, LGPL-2.1-or-later",
    "url": "https://www.7-zip.org/license.txt"
}
```

## URL Fragment Trick

Append `#/dl.7z` to an `.exe` URL to have Scoop download it but treat it as a 7z archive:

```json
"url": "https://example.com/installer.exe#/dl.7z"
```

This bypasses silent-install flags and extracts files directly. Common for MSI/EXE installers.
