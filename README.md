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
