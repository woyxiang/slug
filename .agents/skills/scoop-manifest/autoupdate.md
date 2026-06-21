# Autoupdate & Checkver

> Adapted from [Scoop Wiki - App Manifest Autoupdate](https://github.com/ScoopInstaller/Scoop/wiki/App-Manifest-Autoupdate)

Cross-references: [manifest-schema.md](manifest-schema.md) | [examples.md](examples.md) | [workflow.md](workflow.md)

## checkver

`checkver` tells Scoop how to detect the latest version. Without it, autoupdate won't work.

### Simplest: GitHub shorthand

If the project uses GitHub Releases, set `homepage` to the repo URL and `checkver` to `"github"`:

```json
"homepage": "https://github.com/user/repo",
"checkver": "github"
```

This matches tags like `/releases/tag/(?:v|V)?([\d.]+)`. Pre-releases are ignored.

### Regex on homepage

Match a version pattern on the homepage HTML:

```json
"checkver": "Version ([\\d.]+)"
```

### Regex on a different URL

When the homepage doesn't contain the version:

```json
"checkver": {
    "url": "https://www.7-zip.org/download.html",
    "regex": "Download 7-Zip ([\\d.]+)"
}
```

### JSONPath on JSON endpoint

```json
"checkver": {
    "url": "https://example.com/releases.json",
    "jsonpath": "$.latestVersion"
}
```

### JSONPath + Regex combined

First extract with JSONPath, then match regex on the result:

```json
"checkver": {
    "url": "https://nwjs.io/versions.json",
    "jsonpath": "$.stable",
    "regex": "v([\\d.]+)"
}
```

### Capture groups

Named groups in regex become `$matchX` variables for use in autoupdate URLs:

```json
"checkver": {
    "url": "https://github.com/user/repo/releases/latest",
    "regex": "v(?<version>[\\d\\w.]+)/Tool-(?<short>[\\d.]+).*\\.exe"
}
```

This creates `$matchVersion` and `$matchShort` (or `$match1`, `$match2`).

### Other checkver options

| Property | Description |
|---|---|
| `reverse` | `true` to match the last occurrence instead of first |
| `replace` | Transform the matched value (supports captured variables) |
| `xpath` | XPath expression for XML sources |
| `script` | PowerShell script for complex scenarios |

## autoupdate

`autoupdate` defines URL templates that Scoop updates when a new version is found by `checkver`.

### Basic structure

```json
"autoupdate": {
    "architecture": {
        "64bit": {
            "url": "https://github.com/user/repo/releases/download/$version/app-$version-x64.zip"
        },
        "32bit": {
            "url": "https://github.com/user/repo/releases/download/$version/app-$version-x86.zip"
        }
    }
}
```

If all architectures use the same URL pattern, you can set it globally:

```json
"autoupdate": {
    "url": "https://example.com/app-$version.zip"
}
```

### Properties that can be autoupdated

`url`, `hash`, `bin`, `extract_dir`, `extract_to`, `env_add_path`, `env_set`, `installer`, `license`, `note`, `persist`, `post_install`, `psmodule`, `shortcuts`.

## hash autoupdate

How Scoop gets file hashes during autoupdate (without downloading the file):

### Strategy 1: Sidecar hash file (`$url.sha256`)

Most common for GitHub projects. Scoop appends `.sha256` to the download URL:

```json
"autoupdate": {
    "architecture": {
        "64bit": {
            "url": "https://github.com/user/repo/releases/download/$version/app-$version-x64.zip"
        }
    },
    "hash": {
        "url": "$url.sha256"
    }
}
```

### Strategy 2: Checksum file on same base URL

```json
"hash": {
    "url": "$baseurl/SHA256SUMS"
}
```

Scoop uses built-in regex to find the hash matching `$basename`.

### Strategy 3: Custom regex on checksum file

```json
"hash": {
    "url": "$baseurl/hashes.txt",
    "find": "SHA256\\($basename\\)=\\s+([a-fA-F\\d]{64})"
}
```

### Strategy 4: JSON endpoint

```json
"hash": {
    "mode": "json",
    "jp": "$.files.['$basename'].sha256",
    "url": "$baseurl/hashes.json"
}
```

### Strategy 5: No hash (fallback)

If no `hash` is defined, Scoop downloads the file and hashes it locally. Works but slower.

### Hash modes summary

| Mode | Source | Use When |
|---|---|---|
| `extract` (default) | Plain text / webpage with regex | Checksum file or page |
| `json` | JSON with JSONPath | API provides hashes |
| `xpath` | XML with XPath | XML manifests |
| `rdf` | RDF digest file | e.g. ImageMagick |
| `metalink` | Metalink header/.meta4 | Metalink-enabled mirrors |
| `fosshub` | Auto-detected | FossHub URLs |
| `sourceforge` | Auto-detected | SourceForge URLs |
| `download` | Download + hash locally | Last resort |

## Testing with checkver.ps1

From the bucket repository, use `checkver.ps1` to verify `checkver` works:

```powershell
cd <bucket-repo>

# Check latest version for a single app
.\bin\checkver.ps1 <app>

# Check all apps
.\bin\checkver.ps1 *

# Check and auto-update the manifest
.\bin\checkver.ps1 <app> -u

# Force check (even if version hasn't changed)
.\bin\checkver.ps1 <app> -f

# Check only outdated apps
.\bin\checkver.ps1 <app> -s
```

After running with `-u`, verify that `url`, `hash`, and `extract_dir` in the updated manifest are correct. Then test installation:

```powershell
scoop install bucket\<app>.json
```

## Internal Variables

### Version variables

| Variable | Example (`3.7.1`) | Description |
|---|---|---|
| `$version` | `3.7.1` | Full version |
| `$majorVersion` | `3` | Major |
| `$minorVersion` | `7` | Minor |
| `$patchVersion` | `1` | Patch |
| `$cleanVersion` | `371` | Dots removed |
| `$underscoreVersion` | `3_7_1` | Dots → underscores |
| `$dashVersion` | `3-7-1` | Dots → dashes |
| `$preReleaseVersion` | `rc.1` (from `3.7.1-rc.1`) | After last `-` |

### URL variables

| Variable | Example | Description |
|---|---|---|
| `$url` | `https://example.com/path/file.zip` | Full URL (no fragment) |
| `$baseurl` | `https://example.com/path` | URL without filename |
| `$basename` | `file.zip` | Filename only |

### Captured variables (from checkver regex)

| Variable | Description |
|---|---|
| `$match1`, `$match2`... | Unnamed capture groups |
| `$matchName` | Named group `(?<Name>...)` (capitalize first letter) |
