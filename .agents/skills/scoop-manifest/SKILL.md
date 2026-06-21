---
description: 'Create Scoop application manifests for Windows. Use when: (1) user asks to create a scoop manifest, bucket JSON, or app manifest, (2) user wants to package a CLI tool for scoop, (3) user mentions scoop bucket, (4) user asks how to add an app to scoop'
metadata:
    github-path: skills/scoop-manifest
    github-ref: refs/heads/master
    github-repo: https://github.com/woyxiang/skills
    github-tree-sha: 0a567ff1307a794c31731e950cc28037df7c3894
name: scoop-manifest
---
# Scoop Manifest

Create JSON manifests for the [Scoop](https://scoop.sh) Windows package manager.

## Detection

**Triggers:** "scoop manifest", "bucket JSON", "app manifest", "create manifest for", "add to scoop", "scoop package"

**File patterns:** `bucket/*.json` in Scoop bucket repos

**Keywords:** `scoop`, `manifest`, `bucket`, `checkver`, `autoupdate`

## Routing

| When you need to... | Read |
|---|---|
| Understand the manifest schema (all fields) | [manifest-schema.md](manifest-schema.md) |
| Set up version detection and auto-update | [autoupdate.md](autoupdate.md) |
| Add install/uninstall scripts or persist data | [install-scripts.md](install-scripts.md) |
| Follow the step-by-step workflow to create a manifest | [workflow.md](workflow.md) |
| See real manifest patterns from the Main bucket | [examples.md](examples.md) |

## Workflow Summary

1. **Search** — Find the GitHub repo, identify the latest release
2. **Inspect** — Check release assets for Windows binaries, note naming convention
3. **Analyze** — Download zip, find `bin` path and `extract_dir`
4. **Build** — Fill fields per [manifest-schema.md](manifest-schema.md), pick pattern from [examples.md](examples.md)
5. **Autoupdate** — Set `checkver` + `autoupdate` per [autoupdate.md](autoupdate.md)
6. **Validate** — JSON syntax + structural consistency

## Dependencies

- curl or gh(use api to find GitHub repos and release pages)
- Web search (to find app's home page)
- JSON validation (`node -e "require('./file.json')"` or similar)

## Source

Manifest schema and autoupdate docs adapted from the [Scoop Wiki](https://github.com/ScoopInstaller/Scoop/wiki).
