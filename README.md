# MuxCore Official Spool

This is the official module spool for MuxCore. Add this repository URL to your MuxCore instance to discover official modules and tags.

## URL

```
https://github.com/Muxcore-Media/spool
```

## Tags

Tags are curated module presets — a named set of modules that work together. Pull a tag to load its modules:

```bash
# Start MuxCore with the default tag from the official spool
muxcored --tag default

# Use a minimal preset
muxcored --tag minimal
```

### Available Tags

| Tag | Description |
|-----|-------------|
| `default` | Admin UI, REST API, local auth, memory cache, health monitoring, rate limiting, cron scheduler, workflow engine |
| `minimal` | Bare essentials: admin UI, REST API, local auth, memory cache, health monitoring |

### Tag Definition Format

Tags live in `tags/<name>.json`:

```json
{
  "name": "default",
  "description": "The official MuxCore starter setup",
  "version": "1.0.0",
  "modules": [
    {
      "repo": "https://github.com/Muxcore-Media/admin-ui",
      "version": "v1.0.0",
      "required": true
    }
  ]
}
```

- `required: true` — core refuses to start without this module
- `required: false` — core starts without it but logs a warning

## For Third-Party Spools

Anyone can create a spool. Create a repo with a `catalog.json` and `tags/` directory. Users add your repo URL to their MuxCore instance with `--spool <url>`.

### Security Warning

**Third-party spools are untrusted.** Modules run with the same privileges as the core process. See [SECURITY.md](SECURITY.md) for the full security disclaimer.

## Module Repo Requirements

Each module repo must have a `muxcore.json` at its root with:

```json
{
  "name": "Module Name",
  "description": "What it does",
  "version": "1.0.0",
  "icon": "https://...",
  "author": "Author Name",
  "kind": "downloader",
  "capabilities": ["downloader.torrent"],
  "dependencies": [],
  "homepage": "https://github.com/org/repo"
}
```

### Contract Declarations (Required for Third-Party Modules)

If your module uses contract repos that are not from the `Muxcore-Media` GitHub organization, declare them in `muxcore.json` under `contracts`:

```json
{
  "name": "My Custom Downloader",
  "kind": "downloader",
  "capabilities": ["downloader.custom"],
  "contracts": [
    {
      "repo": "github.com/my-org/contracts-downloader",
      "version": "v2.0.0",
      "interface": "Downloader"
    }
  ]
}
```

#### How Contract Reconciliation Works

MuxCore uses Go's nominal type system for compile-time safety. Two interfaces with identical method sets but different package paths (e.g., `github.com/my-org/contracts-downloader.Downloader` vs `github.com/Muxcore-Media/contracts-downloader.Downloader`) are different types in Go.

When the spool imports a module with non-canonical contracts:

1. The reconciler (`contracts-reconciler`) fetches both the third-party contract repo and the canonical Muxcore-Media equivalent
2. It parses both Go source files and extracts interface method signatures via AST
3. If the method sets are structurally identical, it generates a `go.mod replace` directive that normalizes the import to the canonical path
4. If they differ, the import is rejected with a detailed mismatch report

This preserves compile-time type safety while keeping the MuxCore philosophy: **contracts are patterns, not org-bound dependencies.**

#### Requirements for Contract Repos

Third-party contract repos must define interfaces with **structurally identical method signatures** to the canonical equivalents. Only method names, parameter types, and return types are compared — package paths, comments, and embedded interfaces are ignored.

**Recommended:** Use Go type aliases to avoid reconciliation entirely:

```go
package downloader

import "github.com/Muxcore-Media/contracts-downloader"

// This IS the same type — no reconciliation needed
type Downloader = contracts_downloader.Downloader
```

Modules that use type aliases don't need contract declarations — the Go compiler already sees them as the canonical types.
