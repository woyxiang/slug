# Workflow: Creating a Scoop Manifest

Cross-references: [manifest-schema.md](manifest-schema.md) | [autoupdate.md](autoupdate.md) | [examples.md](examples.md) | [install-scripts.md](install-scripts.md)

## 6-Step Process

### Step 1: Identify the Project

Find the project's download source. It can be any of:

| Source Type | Example | checkver approach |
|---|---|---|
| GitHub Releases | `github.com/user/repo` | `"github"` shorthand |
| Official website | `nmap.org`, `curl.se`, `sqlite.org` | `checkver: { url, regex }` |
| Third-party builds | `nuwen.net` (gcc), `GyanD` (ffmpeg) | `checkver: { url, regex }` |

You need:
- Project description (one line)
- License (SPDX identifier)
- Download URL pattern for Windows binaries

### Step 2: Locate Windows Binaries

**For GitHub projects:** Check the latest release page for assets.

**For non-GitHub projects:** Find the download page on the project's website. Look for a "Downloads" page, "Releases" section, or direct file listing.

Common asset naming patterns:

| Asset Pattern | Meaning |
|---|---|
| `*-x86_64-pc-windows-msvc.zip` | Rust project, 64bit |
| `*-aarch64-pc-windows-msvc.zip` | Rust project, ARM64 |
| `*-i686-pc-windows-msvc.zip` | Rust project, 32bit |
| `*-windows_amd64.zip` | Go project, 64bit |
| `*-windows_arm64.zip` | Go project, ARM64 |
| `*-win64.zip` | Generic 64bit naming |
| `*-win32.zip` | Generic 32bit naming |
| `*-x64.zip` | Generic 64bit naming |
| `*.msi` | MSI installer (extract, don't install) |
| `*.exe` | EXE installer (use `#/dl.7z` trick or `installer`) |

**Identify the naming convention** — you'll need it for `autoupdate` URL templates:
- Version prefix: `v$version`, `$version`, or none
- Arch naming: `x86_64`/`aarch64` (Rust), `amd64`/`arm64` (Go), `win64`/`win32`, `x64`/`x86`

### Step 3: Analyze the Archive

Download and inspect the zip to determine:

1. **`bin` path** — Where is the exe relative to the zip root?
   - Directly in root: `"bin": "program.exe"`
   - In subdirectory: `"bin": "bin\\program.exe"` or `"bin": "subdir\\tool.exe"`

2. **`extract_dir`** — Does the zip contain a top-level directory?
   - Zip contains `program-1.0.0/` with files inside → `"extract_dir": "program-1.0.0"`
   - Zip contains files directly → no `extract_dir` needed

3. **Multiple binaries** — Are there several tools to expose?
   - List all in `"bin": ["tool1.exe", "tool2.exe"]`

### Step 4: Choose a Pattern

Based on what you found, select the closest match from [examples.md](examples.md):

| Situation | Pattern | Example |
|---|---|---|
| GitHub Releases, single arch | Minimal | `just.json` |
| GitHub Releases, multi-arch | Architecture block | `fzf.json` |
| Zip with subdirectory | `extract_dir` | `ripgrep.json` |
| Hash from checksum file | `hash.url` | `gh.json` |
| GitHub with custom tag format | `checkver` with regex | `jq.json` |
| MSI/EXE installer | URL fragment `#/dl.7z` | `7zip.json` |
| Official website download | `checkver: { url, regex }` | `nmap.json`, `curl.json` |
| Non-GitHub custom builds | `checkver: { url, regex }` | `gcc.json` |
| Multiple download files | `url` array | `trid.json` |

### Step 5: Build the Manifest

Construct the JSON following [manifest-schema.md](manifest-schema.md). Key decisions:

1. **`license`** — Look at the repo's LICENSE file. Use SPDX identifier.
2. **`architecture`** — Include all Windows arches the project provides.
3. **`checkver`** — Use `"github"` if the project uses GitHub Releases.
4. **`autoupdate`** — Build URL templates by replacing version with `$version`.
5. **`hash`** — Pick a strategy from [autoupdate.md](autoupdate.md) based on what the project provides.
6. **`suggest`** — Only for genuinely useful companion tools (not hard dependencies).

### Step 6: Validate

1. **JSON syntax** — Run `node -e "require('./file.json'); console.log('Valid')"` or equivalent
2. **Field consistency** — Verify:
   - `version` matches the download URL
   - `bin` path exists in the archive
   - `autoupdate` URL templates produce valid URLs when `$version` is substituted
   - `hash` URL is reachable
   - `checkver` can actually find the version
3. **Test with `checkver.ps1`** — From the bucket repository, run the version checker:
   ```powershell
   cd <bucket-repo>
   .\bin\checkver.ps1 <app>
   ```
   This verifies that `checkver` can find the latest version. If it returns the correct version, update the manifest:
   ```powershell
   .\bin\checkver.ps1 <app> -u
   ```
   Then verify the updated manifest has correct `url`, `hash`, and `extract_dir` values.
4. **Test install** (if Scoop is available):
   ```powershell
   scoop install bucket\<app>.json
   ```

## Quick Decision Tree

```
Where is the project hosted?
├── GitHub Releases
│   ├── homepage = repo URL? → checkver: "github"
│   └── homepage ≠ repo URL? → checkver: { github: "repo URL" }
│   └── Do they publish checksums?
│       ├── Yes → hash.url: "$url.sha256" or "$baseurl/SHA256SUMS"
│       └── No → omit hash (Scoop downloads + hashes locally)
├── Official website (nmap.org, curl.se, sqlite.org, ...)
│   ├── Find a download page with version number
│   ├── Write regex to extract version from that page
│   └── checkver: { url: "download page URL", regex: "version pattern" }
└── Third-party builds (nuwen.net, GyanD, ...)
    └── Same as official website — find a page with version, write regex

Is the download a zip with a subdirectory?
├── Yes → add extract_dir
└── No → no extract_dir needed

Is the download an .exe or .msi installer?
├── Yes → use URL fragment "#/dl.7z" to extract
│   └── Or use installer: { file, args: ["/S"] } for silent install
└── No → direct zip download

Multiple separate download files?
├── Yes → use url array: ["url1", "url2"]
└── No → single url string
```
