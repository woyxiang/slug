# Manifest Examples

> Real patterns from the [Scoop Main bucket](https://github.com/ScoopInstaller/Main/tree/master/bucket)

Cross-references: [manifest-schema.md](manifest-schema.md) | [autoupdate.md](autoupdate.md) | [workflow.md](workflow.md)

## Pattern 1: Minimal (single arch, simple zip)

Source: `just.json`

```json
{
    "version": "1.53.0",
    "description": "A command runner written in rust",
    "homepage": "https://just.systems",
    "license": "CC0-1.0",
    "suggest": {
        "fzf": "fzf"
    },
    "architecture": {
        "64bit": {
            "url": "https://github.com/casey/just/releases/download/1.53.0/just-1.53.0-x86_64-pc-windows-msvc.zip",
            "hash": "02308af0c174d3e0286f6bfc2b020258a3e12cd0c52c1d16fac8ef2c12dea0af"
        }
    },
    "bin": "just.exe",
    "checkver": {
        "github": "https://github.com/casey/just"
    },
    "autoupdate": {
        "architecture": {
            "64bit": {
                "url": "https://github.com/casey/just/releases/download/$version/just-$version-x86_64-pc-windows-msvc.zip"
            }
        }
    }
}
```

**Key points:**
- Only 64bit (project doesn't provide 32bit/arm64)
- `checkver.github` ≠ `homepage` (different URLs)
- No `hash` in autoupdate (Scoop downloads + hashes locally)

---

## Pattern 2: Multi-arch zip

Source: `fzf.json`

```json
{
    "version": "0.73.1",
    "description": "A general-purpose command-line fuzzy finder",
    "homepage": "https://github.com/junegunn/fzf",
    "license": "MIT",
    "architecture": {
        "64bit": {
            "url": "https://github.com/junegunn/fzf/releases/download/v0.73.1/fzf-0.73.1-windows_amd64.zip",
            "hash": "521a974dc32e93404265e55bffaf71a59e05e80abdf8ca4afb21a6030dc76f5f"
        },
        "arm64": {
            "url": "https://github.com/junegunn/fzf/releases/download/v0.73.1/fzf-0.73.1-windows_arm64.zip",
            "hash": "1b043fd2eea42b210679be71a15c0be9516a4369ce5920f533e6b02edb8736b9"
        }
    },
    "bin": "fzf.exe",
    "checkver": "github",
    "autoupdate": {
        "architecture": {
            "64bit": {
                "url": "https://github.com/junegunn/fzf/releases/download/v$version/fzf-$version-windows_amd64.zip"
            },
            "arm64": {
                "url": "https://github.com/junegunn/fzf/releases/download/v$version/fzf-$version-windows_arm64.zip"
            }
        }
    }
}
```

**Key points:**
- Go naming convention: `windows_amd64`, `windows_arm64`
- `checkver: "github"` (homepage = repo URL)
- No `hash` in autoupdate

---

## Pattern 3: Zip with extract_dir

Source: `ripgrep.json`

```json
{
    "version": "15.1.0",
    "description": "Recursively searches directories for a regex pattern.",
    "homepage": "https://github.com/BurntSushi/ripgrep",
    "license": "MIT",
    "suggest": {
        "vcredist": "extras/vcredist2022"
    },
    "architecture": {
        "64bit": {
            "url": "https://github.com/BurntSushi/ripgrep/releases/download/15.1.0/ripgrep-15.1.0-x86_64-pc-windows-msvc.zip",
            "hash": "124510b94b6baa3380d051fdf4650eaa80a302c876d611e9dba0b2e18d87493a",
            "extract_dir": "ripgrep-15.1.0-x86_64-pc-windows-msvc"
        },
        "32bit": {
            "url": "https://github.com/BurntSushi/ripgrep/releases/download/15.1.0/ripgrep-15.1.0-i686-pc-windows-msvc.zip",
            "hash": "725be85a1e8f92878a548f40ee4f6df64bc93b809586462b3c6d884e1de1e83a",
            "extract_dir": "ripgrep-15.1.0-i686-pc-windows-msvc"
        },
        "arm64": {
            "url": "https://github.com/BurntSushi/ripgrep/releases/download/15.1.0/ripgrep-15.1.0-aarch64-pc-windows-msvc.zip",
            "hash": "00d931fb5237c9696ca49308818edb76d8eb6fc132761cb2a1bd616b2df02f8e",
            "extract_dir": "ripgrep-15.1.0-aarch64-pc-windows-msvc"
        }
    },
    "bin": "rg.exe",
    "checkver": "github",
    "autoupdate": {
        "architecture": {
            "64bit": {
                "url": "https://github.com/BurntSushi/ripgrep/releases/download/$version/ripgrep-$version-x86_64-pc-windows-msvc.zip",
                "extract_dir": "ripgrep-$version-x86_64-pc-windows-msvc"
            },
            "32bit": {
                "url": "https://github.com/BurntSushi/ripgrep/releases/download/$version/ripgrep-$version-i686-pc-windows-msvc.zip",
                "extract_dir": "ripgrep-$version-i686-pc-windows-msvc"
            },
            "arm64": {
                "url": "https://github.com/BurntSushi/ripgrep/releases/download/$version/ripgrep-$version-aarch64-pc-windows-msvc.zip",
                "extract_dir": "ripgrep-$version-aarch64-pc-windows-msvc"
            }
        }
    }
}
```

**Key points:**
- Zip contains `ripgrep-15.1.0-x86_64-pc-windows-msvc/` subdirectory → `extract_dir`
- `extract_dir` must also be in `autoupdate` (with `$version`)
- `suggest` for vcredist (common for Rust projects)

---

## Pattern 4: Hash from checksum file

Source: `gh.json`

```json
{
    "version": "2.95.0",
    "description": "Official GitHub CLI",
    "homepage": "https://cli.github.com",
    "license": "MIT",
    "architecture": {
        "64bit": {
            "url": "https://github.com/cli/cli/releases/download/v2.95.0/gh_2.95.0_windows_amd64.zip",
            "hash": "19a7154161ada9cfaa9e57edb752ecc679b75c391a62e4f7b586eea1df30b5bb"
        },
        "32bit": {
            "url": "https://github.com/cli/cli/releases/download/v2.95.0/gh_2.95.0_windows_386.zip",
            "hash": "6fd704aedfb00eb8f0db49d4f22084ffabe7b8eb1212f5bf52fa2dbd52e7c50c"
        },
        "arm64": {
            "url": "https://github.com/cli/cli/releases/download/v2.95.0/gh_2.95.0_windows_arm64.zip",
            "hash": "f7df0bf24275196b0fcbe853946e53ae80121c8a4ce4341bc9de1a35e33b4588"
        }
    },
    "bin": "bin\\gh.exe",
    "checkver": {
        "github": "https://github.com/cli/cli"
    },
    "autoupdate": {
        "architecture": {
            "64bit": {
                "url": "https://github.com/cli/cli/releases/download/v$version/gh_$version_windows_amd64.zip"
            },
            "32bit": {
                "url": "https://github.com/cli/cli/releases/download/v$version/gh_$version_windows_386.zip"
            },
            "arm64": {
                "url": "https://github.com/cli/cli/releases/download/v$version/gh_$version_windows_arm64.zip"
            }
        },
        "hash": {
            "url": "$baseurl/gh_$version_checksums.txt"
        }
    }
}
```

**Key points:**
- Binary is in `bin\` subdirectory → `"bin": "bin\\gh.exe"`
- Hash from checksum file: `gh_$version_checksums.txt`
- `$baseurl` = download base URL without filename

---

## Pattern 5: Sidecar hash file (`$url.sha256`)

Source: `starship.json`

```json
{
    "version": "1.25.1",
    "description": "The minimal, blazing fast, and extremely customizable prompt for any shell!",
    "homepage": "https://starship.rs",
    "license": "ISC",
    "suggest": {
        "vcredist": "extras/vcredist2022"
    },
    "architecture": {
        "64bit": {
            "url": "https://github.com/starship/starship/releases/download/v1.25.1/starship-x86_64-pc-windows-msvc.zip",
            "hash": "a07cf3e428afab09324e510fb786041ebcc491a68b1ca6fba044c5a461f9b017"
        },
        "32bit": {
            "url": "https://github.com/starship/starship/releases/download/v1.25.1/starship-i686-pc-windows-msvc.zip",
            "hash": "db9a91315ee9c247a76604c160dbd8e7d40fb8931fd90070b9c20c0c1ff883c0"
        },
        "arm64": {
            "url": "https://github.com/starship/starship/releases/download/v1.25.1/starship-aarch64-pc-windows-msvc.zip",
            "hash": "d5b3598cb1392be74508b5e5fe3583cbcad307e7788dad804c7bf2e9ab84b663"
        }
    },
    "bin": "starship.exe",
    "checkver": {
        "github": "https://github.com/starship/starship"
    },
    "autoupdate": {
        "architecture": {
            "64bit": {
                "url": "https://github.com/starship/starship/releases/download/v$version/starship-x86_64-pc-windows-msvc.zip"
            },
            "32bit": {
                "url": "https://github.com/starship/starship/releases/download/v$version/starship-i686-pc-windows-msvc.zip"
            },
            "arm64": {
                "url": "https://github.com/starship/starship/releases/download/v$version/starship-aarch64-pc-windows-msvc.zip"
            }
        },
        "hash": {
            "url": "$url.sha256"
        }
    }
}
```

**Key points:**
- `$url.sha256` — Scoop appends `.sha256` to the download URL
- Cleanest pattern for projects that publish sidecar hash files

---

## Pattern 6: Custom checkver (regex on different URL)

Source: `jq.json`

```json
{
    "version": "1.8.2",
    "description": "Lightweight and flexible command-line JSON processor",
    "homepage": "https://jqlang.github.io/jq/",
    "license": "MIT",
    "suggest": {
        "jid": "jid"
    },
    "architecture": {
        "64bit": {
            "url": "https://github.com/jqlang/jq/releases/download/jq-1.8.2/jq-windows-amd64.exe#/jq.exe",
            "hash": "a6fc67fedaf9128a3309a1e2ebb8b986aeccf70122ee46d2cb4849e423f0c627"
        },
        "32bit": {
            "url": "https://github.com/jqlang/jq/releases/download/jq-1.8.2/jq-windows-i386.exe#/jq.exe",
            "hash": "a99cb668f95bdd788d9ee20529613b115e5d2a0d7f9127ee6976607e878558ba"
        }
    },
    "bin": "jq.exe",
    "checkver": {
        "github": "https://github.com/jqlang/jq",
        "regex": "jq-([\\d.]+)"
    },
    "autoupdate": {
        "architecture": {
            "64bit": {
                "url": "https://github.com/jqlang/jq/releases/download/jq-$version/jq-windows-amd64.exe#/jq.exe"
            },
            "32bit": {
                "url": "https://github.com/jqlang/jq/releases/download/jq-$version/jq-windows-i386.exe#/jq.exe"
            }
        },
        "hash": {
            "url": "https://github.com/jqlang/jq/releases/download/jq-$version/sha256sum.txt"
        }
    }
}
```

**Key points:**
- `homepage` ≠ repo URL → `checkver.github` points to repo separately
- Custom `regex` because tag format is `jq-1.8.2` not `v1.8.2`
- URL fragment `#/jq.exe` renames the downloaded `.exe` (force Scoop to treat it as a file, not installer)
- Hash from `sha256sum.txt` in the same release

---

## Pattern 7: Persist + env_set

Source: `bat.json`

```json
{
    "version": "0.26.1",
    "description": "A cat(1) clone with syntax highlighting and Git integration",
    "homepage": "https://github.com/sharkdp/bat",
    "license": "Apache-2.0",
    "suggest": {
        "vcredist": "extras/vcredist2022",
        "less": "less"
    },
    "architecture": {
        "64bit": {
            "url": "https://github.com/sharkdp/bat/releases/download/v0.26.1/bat-v0.26.1-x86_64-pc-windows-msvc.zip",
            "hash": "...",
            "extract_dir": "bat-v0.26.1-x86_64-pc-windows-msvc"
        }
    },
    "pre_install": [
        "if (!(Test-Path \"$persist_dir\\config\")) {",
        "    Copy-Item -ErrorAction Ignore \"$env:APPDATA\\bat\\config\" \"$dir\\config\"",
        "    New-Item -ErrorAction Ignore \"$dir\\config\" | Out-Null",
        "}"
    ],
    "env_set": {
        "BAT_CONFIG_DIR": "$dir"
    },
    "persist": ["config", "syntaxes", "themes"],
    "bin": "bat.exe",
    "checkver": "github",
    "autoupdate": { "..." : "..." }
}
```

**Key points:**
- `persist` keeps config/themes across updates
- `env_set` points the app to the install dir for config
- `pre_install` migrates existing user config on first install

---

## Pattern 8: Multiple binaries

Source: `nmap.json`

```json
{
    "bin": [
        "nmap.exe",
        "ncat.exe",
        "ndiff.bat",
        "nping.exe"
    ]
}
```

Source: `helix.json` (alias):

```json
{
    "bin": [
        "hx.exe",
        ["hx.exe", "helix"]
    ]
}
```

This adds both `hx` and `helix` to PATH, both pointing to `hx.exe`.

---

## Pattern 9: Official website download (non-GitHub)

Source: `nmap.json`

```json
{
    "version": "7.99",
    "description": "Network exploration and security auditing utility.",
    "homepage": "https://nmap.org",
    "license": {
        "identifier": "GPL-2.0-only",
        "url": "https://github.com/nmap/nmap/blob/master/COPYING"
    },
    "url": "https://nmap.org/dist/nmap-7.99-setup.exe#/dl.7z",
    "hash": "fda839f35d9f8f18a11670e17d0332ce9d05a3556c5a20e91b0b56c57774f611",
    "bin": ["nmap.exe", "ncat.exe", "ndiff.bat", "nping.exe"],
    "checkver": {
        "url": "https://nmap.org/download.html",
        "regex": "nmap-([\\d.]+)-setup\\.exe"
    },
    "autoupdate": {
        "url": "https://nmap.org/dist/nmap-$version-setup.exe#/dl.7z"
    }
}
```

**Key points:**
- Homepage is the project website, not GitHub
- `checkver` fetches a custom download page and uses regex to find version
- EXE installer with `#/dl.7z` trick to extract silently
- No `architecture` block — single URL for all arches

---

## Pattern 10: Non-GitHub third-party builds

Source: `gcc.json`

```json
{
    "version": "15.2.0",
    "description": "GNU Compiler Collection and binutils",
    "homepage": "https://www.gnu.org/software/gcc/",
    "license": "GPL-3.0-or-later",
    "architecture": {
        "64bit": {
            "url": "https://nuwen.net/files/mingw/components-20.0.7z",
            "hash": "561d873b7f95dbb39a34b7ab00050dc6028808310a847721a8aea5e5b0bff1c9"
        }
    },
    "extract_dir": "components-20.0",
    "pre_install": [
        "Expand-7ZipArchive \"$dir\\binutils-*.7z\" \"$dir\"",
        "Expand-7ZipArchive \"$dir\\mingw-w64+gcc.7z\" \"$dir\"",
        "Get-ChildItem \"$dir\\*.7z\" | Remove-Item -Recurse -Force"
    ],
    "env_add_path": "bin",
    "env_set": {
        "C_INCLUDE_PATH": "$dir\\include",
        "CPLUS_INCLUDE_PATH": "$dir\\include"
    },
    "checkver": {
        "url": "https://nuwen.net/mingw.html",
        "regex": "(?sm)>GCC ([\\d.]+)</a>.*?components-(?<components>[\\d.]+).7z"
    },
    "autoupdate": {
        "architecture": {
            "64bit": {
                "url": "https://nuwen.net/files/mingw/components-$matchComponents.7z"
            }
        },
        "extract_dir": "components-$matchComponents"
    }
}
```

**Key points:**
- Downloaded from a third-party mirror (nuwen.net), not the official GCC site
- `checkver` regex uses capture groups (`?<components>`) to build version-specific URLs
- `pre_install` extracts nested 7z archives
- `extract_dir` uses `$matchComponents` variable from checkver

---

## Pattern 11: Multiple download files

Source: `trid.json`

```json
{
    "version": "2.48-26.06.19",
    "description": "A utility designed to identify file types from their binary signatures.",
    "homepage": "https://www.mark0.net/soft-trid-e.html",
    "license": "Shareware",
    "architecture": {
        "64bit": {
            "url": [
                "https://www.mark0.net/download/triddefs.zip",
                "https://www.mark0.net/download/trid_win64.zip"
            ],
            "hash": [
                "a5c41dea17b9bda7198d212a04322f89583da9f311cf875c934f635b778f2923",
                "65d339cc3f758a9d78e41f2c418955dfcd8c4a5152682e8cc9af0f551d169a4b"
            ]
        }
    },
    "bin": "trid.exe",
    "checkver": {
        "regex": "(?si)Win/x86-64.+?TrID v?([\\d.]+).+?(?<day>\\d{2})/(?<month>\\d{2})/(?<year>\\d+)",
        "replace": "${1}-${year}.${month}.${day}"
    },
    "autoupdate": {
        "architecture": {
            "64bit": {
                "url": [
                    "https://www.mark0.net/download/triddefs.zip",
                    "https://www.mark0.net/download/trid_win64.zip"
                ]
            }
        }
    }
}
```

**Key points:**
- Two separate downloads (definitions + binary) → `url` and `hash` are arrays
- `checkver.replace` constructs a composite version from captured groups
- Non-GitHub, non-standard download page

---

## Pattern 12: Official website with checksum file

Source: `curl.json`

```json
{
    "version": "8.20.0_5",
    "description": "Command line tool and library for transferring data with URLs",
    "homepage": "https://curl.se/",
    "license": "MIT",
    "architecture": {
        "64bit": {
            "url": "https://curl.se/windows/dl-8.20.0_5/curl-8.20.0_5-win64-mingw.tar.xz",
            "hash": "6a3c78b52d6ee1deb8e1b61d4de60bea08d4c9a18ba0de35507337dc453401cd",
            "extract_dir": "curl-8.20.0_5-win64-mingw"
        },
        "arm64": {
            "url": "https://curl.se/windows/dl-8.20.0_5/curl-8.20.0_5-win64a-mingw.tar.xz",
            "hash": "5e46921ccdbc43279e32a46f85acd53598e9e2d11b235cfc5a16bcfec957c9d5",
            "extract_dir": "curl-8.20.0_5-win64a-mingw"
        }
    },
    "bin": "bin\\curl.exe",
    "checkver": {
        "url": "https://curl.se/windows/",
        "regex": "Build<\\/b>:\\s+([\\d._]+)"
    },
    "autoupdate": {
        "architecture": {
            "64bit": {
                "url": "https://curl.se/windows/dl-$version/curl-$version-win64-mingw.tar.xz",
                "extract_dir": "curl-$version-win64-mingw"
            },
            "arm64": {
                "url": "https://curl.se/windows/dl-$version/curl-$version-win64a-mingw.tar.xz",
                "extract_dir": "curl-$version-win64a-mingw"
            }
        },
        "hash": {
            "url": "$baseurl/hashes.txt",
            "regex": "SHA2-256\\($basename\\)=\\s+$sha256"
        }
    }
}
```

**Key points:**
- Official project website (curl.se), not GitHub
- `checkver` fetches a dedicated Windows build page
- Hash from a `hashes.txt` file with custom regex
- Version in URL path (`dl-$version/`) not just filename

---

## Pattern Comparison: Source Types

| Source | checkver | Example |
|---|---|---|
| GitHub Releases (homepage = repo) | `"github"` | `fzf.json`, `ripgrep.json` |
| GitHub Releases (homepage ≠ repo) | `{"github": "repo URL"}` | `jq.json`, `ffmpeg.json` |
| Official website | `{"url": "download page", "regex": "..."}` | `nmap.json`, `curl.json`, `sqlite.json` |
| Third-party builds | `{"url": "build page", "regex": "..."}` | `gcc.json` |
| Custom download page | `{"regex": "..."}` (on homepage) | `trid.json` |

## Pattern Comparison: Hash Strategies

| Strategy | autoupdate.hash | When to use |
|---|---|---|
| Sidecar file | `{"url": "$url.sha256"}` | Project publishes `<file>.sha256` alongside downloads |
| Checksum file | `{"url": "$baseurl/SHA256SUMS"}` | Single checksum file for all assets |
| Custom regex | `{"url": "...", "find": "regex"}` | Non-standard checksum format |
| JSON API | `{"mode": "json", "jp": "jsonpath", "url": "..."}` | API endpoint provides hashes |
| No hash | (omit) | Fallback — Scoop downloads + hashes locally |
