# Repository Structure

This document describes the file organization of the tunnel-gateway repository.

## Directory Layout

```
tunnel-gateway/
│
├── gtwy-tool/                     # Server-side gateway manager
│   ├── gtwy                       # Main executable (Python script)
│   └── README.md                  # gtwy documentation
│
├── tnl-tool/                      # Client-side tunnel manager
│   ├── tnl                        # Main executable (Python script)
│   └── README.md                  # tnl documentation
│
├── docs/                          # Shared documentation
│   ├── TESTING.md                 # Complete testing guide
│   └── CLAUDE.md                  # Technical specification
│
├── examples/                      # Example configurations
│   ├── config.yml.example         # gtwy configuration template
│   ├── nginx-template             # nginx server block template
│   └── systemd/
│       └── tnl-admin.service      # Example systemd service
│
├── README.md                      # Main project README (root)
├── CHANGELOG.md                   # Version history and changes
├── LICENSE                        # MIT License
├── VERSION                        # Current version number
├── .gitignore                     # Git ignore rules
└── requirements.txt               # Python dependencies (gtwy only)
```

## Files to Include in Git

### Core Files
- `gtwy-tool/gtwy` - Server tool executable
- `tnl-tool/tnl` - Client tool executable
- Both are single-file Python scripts, no external dependencies on templates

### Documentation
- `README.md` - Main project overview (root)
- `gtwy-tool/README.md` - Server tool documentation
- `tnl-tool/README.md` - Client tool documentation
- `docs/TESTING.md` - Testing guide
- `docs/CLAUDE.md` - Technical specification
- `CHANGELOG.md` - Version history

### Examples
- `examples/config.yml.example` - Configuration template
- `examples/nginx-template` - nginx template reference
- `examples/systemd/tnl-admin.service` - systemd service example

### Meta Files
- `LICENSE` - MIT License
- `VERSION` - Version number (1.0.0)
- `.gitignore` - Git ignore patterns
- `requirements.txt` - Python dependencies

## Files Excluded from Git

These files are listed in `.gitignore`:

### Sensitive
- `config.yml` - Contains API keys
- `**/api-keys.md` - API credentials
- `**/secrets.yml` - Any secrets
- `.env*` - Environment files

### Generated/Runtime
- `tunnels.db` - SQLite database
- `*.log` - Log files
- `__pycache__/` - Python cache
- `*.pyc` - Compiled Python

### Local/IDE
- `.DS_Store` - macOS
- `.claude/` - Claude Code config
- `.vscode/` - VS Code settings
- `.idea/` - PyCharm settings

### Old/Drafts
- `documentation/` - Internal notes and concepts
- `legacy/` - Old implementations

## Installation Workflow

### For Gateway Server

1. Download `gtwy-tool/gtwy`
2. Run `sudo ./gtwy install` (one-time setup)
3. Run `sudo gtwy setup` (interactive config)
4. Use example `config.yml.example` as reference

### For IT.Box Client

1. Download `tnl-tool/tnl`
2. Run `sudo ./tnl install` (one-time setup)
3. Run `sudo tnl setup <ip> <port>` (connect to gateway)
4. Systemd service created automatically

## Versioning

- Both `gtwy` and `tnl` share the same version number
- Version is stored in `VERSION` file
- Version is embedded in both scripts as `VERSION = "1.0.0"`
- Git tags follow `vX.Y.Z` format (e.g., `v1.0.0`)

## Release Process

1. Update `VERSION` file
2. Update version in `gtwy` script
3. Update version in `tnl` script
4. Update `CHANGELOG.md`
5. Commit changes: `git commit -am "Release vX.Y.Z"`
6. Tag: `git tag -a vX.Y.Z -m "Release vX.Y.Z"`
7. Push: `git push && git push --tags`
8. Create GitHub Release with `gtwy` and `tnl` as downloadable assets

## Download URLs (after release)

Users can download directly:

```bash
# Gateway tool
curl -O https://github.com/USERNAME/tunnel-gateway/releases/latest/download/gtwy

# Client tool
curl -O https://github.com/USERNAME/tunnel-gateway/releases/latest/download/tnl
```

## Notes

- Both tools are self-contained single-file Python scripts
- nginx template is embedded in `gtwy` as a constant
- No build process required - scripts are ready to use
- Configuration is created during `setup` command
- All runtime state stored in `/opt/gtwy/` (server) or managed by systemd (client)
