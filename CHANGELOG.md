# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.0.0] - 2025-12-28

### Major Release - Repository Reorganization & New Tools

**Breaking Changes:**
- Repository renamed from "tnld" to "IT.Box Tools"
- All tools now at unified version 2.0.0
- Repository structure completely reorganized

### Added

#### New Tools

**mnchk** (v2.0.0) - moin.schule Credential Checker
- Validates username/password against moin.schule SSO
- Multiple password input methods (interactive, stdin, file)
- JSON output support
- Exit codes for script integration
- Self-installing with `sudo mnchk install`

**boxctl** (v2.0.0) - IT.Box Container Management
- Docker container lifecycle management for IT.Box
- Automatic docker-compose.yml generation
- Secrets management with auto-generation
- Container catalog synchronization
- Interactive configuration prompts
- Port conflict detection
- Dependency management
- Self-installing with `sudo boxctl install`

**catalogctl** (v2.0.0) - Container Catalog Management
- Container catalog server management
- YAML validation for container definitions
- Port conflict checking across catalog
- nginx configuration generation for catalog server
- SSL setup with certbot integration
- IONOS DNS integration
- Self-installing with `sudo catalogctl install`

#### Documentation

- New central README.md with tool overview and architecture diagram
- Individual READMEs for all tools in subdirectories
- CLAUDE-CONTAINER-YAML-GUIDE.md for container definition format
- Example container configuration (minecraft.yaml)

### Changed

#### Repository Structure

**Before (v1.2.6):**
```
tnld/
├── README.md (tnld overview)
├── gtwy/
│   ├── gtwy
│   └── README.md
└── tnl/
    ├── tnl
    └── README.md
```

**After (v2.0.0):**
```
IT.Box Tools/
├── README.md (central overview)
├── gtwy/ (v2.0.0)
│   ├── gtwy
│   └── README.md
├── tnl/ (v2.0.0)
│   ├── tnl
│   └── README.md
├── mnchk/ (v2.0.0 - NEW)
│   ├── mnchk
│   └── README.md
├── boxctl/ (v2.0.0 - NEW)
│   ├── boxctl
│   ├── README.md
│   ├── CLAUDE-CONTAINER-YAML-GUIDE.md
│   └── example config - minecraft.yaml
└── catalogctl/ (v2.0.0 - NEW)
    ├── catalogctl
    └── README.md
```

#### Version Unification

- **gtwy**: v1.2.6 → v2.0.0
- **tnl**: v1.2.6 → v2.0.0
- **mnchk**: v1.0.1 → v2.0.0 (incorporated)
- **boxctl**: v0.1.0 → v2.0.0 (incorporated)
- **catalogctl**: v0.1.0 → v2.0.0 (incorporated)

All tools now follow unified versioning scheme.

#### Documentation

- Central README.md: Shortened to overview with tool comparison table
- Tool-specific details moved to individual READMEs
- Added architecture diagram showing all tools
- Added typical workflow section

### Technical

- All tools syntactically validated (Python compile check)
- No compatibility conflicts between tools
- Each tool remains self-contained (single-file)
- Unified installation pattern: `sudo <tool> install`
- Unified version command: `<tool> version`

### Migration Notes (v1.2.6 → v2.0.0)

**For existing tnld users:**
- Repository renamed on GitHub (redirects will work)
- gtwy and tnl functionality unchanged (backward compatible)
- Update tools as usual:
  ```bash
  # Gateway
  curl -LO https://github.com/steedalot/itbox-tools/releases/latest/download/gtwy
  chmod +x gtwy
  sudo ./gtwy update

  # Box
  curl -LO https://github.com/steedalot/itbox-tools/releases/latest/download/tnl
  chmod +x tnl
  sudo ./tnl update
  ```

**For new users:**
- Start with central README.md for overview
- Each tool has comprehensive documentation in its subdirectory
- All tools can be used independently

### Scope

This release marks the expansion from a pure tunnel management system (gtwy + tnl) to a comprehensive IT.Box infrastructure toolkit:

- **Tunneling**: gtwy + tnl (SSH reverse tunnels, DNS, SSL)
- **Container Management**: boxctl + catalogctl (Docker, catalog server)
- **Authentication**: mnchk (moin.schule SSO integration)

---

## Previous Releases (tnld)

The following releases were part of the previous "tnld" repository focusing on tunnel management.

## [1.2.6] - 2025-12-19

### Fixed

#### gtwy (Gateway Manager)
- **Automatic tunnel finalization** - Fixed background SSL certificate provisioning
  - Log file permissions now correctly set to root:gtwy-admin 664
  - Fixed argument order in subprocess call (--config/--db before subcommand)
  - Background process now properly logs to /opt/gtwy/gtwy.log
  - SSH command logging now works (tunneluser can write to log file)

- **DNS record deletion** - Fixed DNS cleanup using correct IONOS API endpoint
  - Changed from non-existent GET /zones/{id}/records to GET /zones/{id}
  - Uses query parameters (recordName, recordType) to filter records
  - DELETE /zones/{id}/records/{recordId} now works correctly
  - DNS records are properly cleaned up when tunnels are removed

#### tnl (Tunnel Client)
- **Zero-downtime updates** - Services continue running during updates
  - Removed service stop/restart from update process
  - No SSH tunnel interruption when updating
  - Update now safe to run over SSH tunnel
  - Only /usr/local/bin/tnl executable is replaced

### Changed

#### gtwy (Gateway Manager)
- Automatic log file setup with correct permissions during install/update
- Installation: Creates log file with root:gtwy-admin 664 (step 11)
- Update: Fixes log file permissions if incorrect (step 9)
- Removed all debug logging statements

### Technical

- Log file accessible by tunneluser via gtwy-admin group membership
- Background finalization runs detached with proper logging
- DNS API uses correct endpoints per official IONOS DNS API spec
- Content-Type header added to all IONOS API requests for consistency
- Services use autossh directly, not tnl binary (enables zero-downtime updates)
- Added version 1.2.6 to migration sequences

### Impact

- ✅ Tunnels now automatically get SSL certificates within ~10 seconds
- ✅ DNS records are properly cleaned up when tunnels are removed
- ✅ All operations are properly logged for debugging
- ✅ Updates can be performed without service interruption

## [1.2.4] - 2025-12-15

### Added

#### gtwy (Gateway Manager)
- **Automatic certificate renewal** - Added certbot post-renewal hook
  - Automatically reloads nginx after Let's Encrypt renews certificates (every 90 days)
  - Hook installed during `gtwy install` and `gtwy update`
  - Located at `/etc/letsencrypt/renewal-hooks/post/nginx-reload.sh`
  - Ensures nginx picks up renewed certificates without manual intervention
  - Prevents HTTPS errors after certificate expiration

### Technical

- New function `setup_certbot_renewal_hook()` creates bash script in certbot's post-renewal hooks directory
- Hook executes `nginx -s reload` after any successful certificate renewal
- Integrated into both installation and update workflows
- Version 1.2.4 added to migration version sequences

## [1.2.3] - 2025-12-15

### Fixed

#### gtwy (Gateway Manager)
- **Critical DNS API fix** - DNS records can now be created successfully
  - Changed IONOS API payload to use full FQDN instead of hostname only
  - Fixed `create_dns_record()`: uses `subdomain` (FQDN) instead of `hostname` in API request
  - Fixed `delete_dns_record()`: matches records by full FQDN instead of hostname
  - Multi-level subdomains like `service.box.domain.tld` now work correctly
  - Improved error logging to show detailed API error responses

### Technical

- IONOS DNS API expects the full FQDN (e.g., `langflow.itbox1.kibox.online`) in the `name` field, not just the hostname part (e.g., `langflow.itbox1`)
- Removed unused `hostname` variable from DNS functions
- Added version 1.2.3 to migration version sequences

## [1.2.2] - 2025-12-14

### Changed

#### gtwy (Gateway Manager)
- **Async certificate workflow** - Major improvement to tunnel creation
  - Tunnel creation now returns immediately (no waiting for DNS/certificate)
  - Certificate requested in background via detached subprocess
  - New `gtwy finalize-tunnel` command (internal, runs in background)
  - DNS propagation check: smart loop, max 120s timeout
  - Tunnels start with HTTP-only, upgrade to HTTPS automatically
  - Status tracking: `pending_cert` → `active` (or `error` on failure)

- **nginx template improvements**
  - Two templates: HTTP-only (pending_cert) and SSL (active)
  - Fixed deprecated `listen 443 ssl http2` → `listen 443 ssl; http2 on;`
  - Inline SSL settings (no dependency on `/etc/letsencrypt/options-ssl-nginx.conf`)
  - Modern TLS configuration: TLSv1.2/1.3, secure ciphers

- **Simplified update paths**
  - Only supports migration from last stable (1.2.1) → current
  - Removed legacy migration code for v1.0.0, v1.1.0, v1.2.0
  - Cleaner, more maintainable codebase

#### tnl (Tunnel Client)
- **Simplified update paths**
  - Only supports migration from last stable (1.2.1) → current
  - Removed legacy migration code

### Added

#### gtwy (Gateway Manager)
- Added `import time` for DNS propagation wait loops
- New `cmd_finalize_tunnel` command for async certificate requests
- `NGINX_TEMPLATE_HTTP` - HTTP-only template for pending certificates
- `NGINX_TEMPLATE_SSL` - SSL template for active tunnels

### Fixed

#### gtwy (Gateway Manager)
- Fixed `request_certificate()` call in `cmd_finalize_tunnel` (missing config parameter)

### Technical
- Background process uses `subprocess.Popen()` with `start_new_session=True`
- Detached from SSH session, survives connection close
- Logging to gtwy.log for background finalization process
- Error handling: failures set tunnel status to 'error' with message
- Smart DNS check: 24 attempts × 5s = 120s max wait

### Workflow

**Before (v1.2.1):**
```
tnl add → gtwy request → DNS + wait + certbot + nginx → return port (slow, 30-120s)
```

**After (v1.2.2):**
```
tnl add → gtwy request → DNS + HTTP nginx → return port (fast, <5s)
           ↓
   [background] gtwy finalize-tunnel → wait DNS → certbot → HTTPS nginx
```

**Benefits:**
- ✅ Instant response to client
- ✅ Service available immediately (HTTP)
- ✅ Automatic upgrade to HTTPS
- ✅ Better error handling
- ✅ No SSH timeout issues

## [1.2.1] - 2025-12-14

### Fixed

#### tnl (Tunnel Client)
- **v1.0.0 → v1.2.x update path** - Fixed update failure for v1.0.0 installations
  - v1.0.0 didn't have `/etc/tnl/config.yml` (introduced in v1.1.0)
  - Update now checks for systemd service OR SSH key instead of just config file
  - Automatic config migration: parses systemd service to create `config.yml`
  - Extracts gateway IP, admin_port, ssh_port from existing service file
  - Seamless upgrade from v1.0.0 without manual intervention

- **Missing shutil import** - Fixed import error in tnl update
  - `shutil` was imported locally in functions instead of globally
  - Caused "cannot access local variable 'shutil'" error during update
  - Now imported at module level

#### gtwy (Gateway Manager)
- **Configuration validation** - Changed from errors to warnings
  - Old configs with different structure no longer fail validation
  - Missing recommended keys now show as warnings, not errors
  - Non-critical warnings don't prevent successful update
  - Clear messaging: "These are non-critical, gtwy should work normally"

- **SSH command restriction** - Fixed authorized_keys for service tunnels
  - Changed from hardcoded `gtwy request` to `gtwy $SSH_ORIGINAL_COMMAND`
  - Allows boxes to call both `request` and `release` with arguments
  - Path updated from `/opt/gtwy/gtwy` to `/usr/local/bin/gtwy`

- **Logging permissions** - Fixed permission error for SSH commands
  - SSH commands (via tunneluser) now use console-only logging
  - No longer tries to write to `/opt/gtwy/gtwy.log` (permission denied)
  - Regular admin commands still use file logging

- **Database permissions** - Fixed readonly database error for SSH commands
  - `init_database()` now automatically sets correct permissions
  - Directory `/opt/gtwy`: root:gtwy-admin 775
  - Database file: tunneluser:gtwy-admin 664
  - Allows tunneluser to write via SSH (request/release commands)

- **nginx config permissions** - Fixed permission denied when writing configs
  - Configs now written to `/opt/gtwy/nginx-configs/` (tunneluser writable)
  - Directory: root:gtwy-admin 775
  - Symlink: `/etc/nginx/sites-enabled/tunnels-autogen.conf` → `/opt/gtwy/nginx-configs/tunnels-autogen.conf`
  - tunneluser (via SSH) can now write nginx configurations

### Technical
- Added v1.2.1 to version sequences in migration runners
- Added config migration framework (similar to database migrations)
- `run_config_migrations()` - Automatic config updates during upgrade
- `migrate_config_1_2_0_to_1_2_1()` - Updates nginx.config_path automatically
- Improved error handling for config parsing edge cases
- Better user feedback during update process
- Console-only logging for SSH-invoked commands (BOX_ID env var detection)
- Update command now creates nginx configs directory and symlink automatically

## [1.2.0] - 2025-12-14

### Added

#### gtwy (Gateway Manager)
- **`update` command** - Update gtwy to new version while preserving configuration
  - Automatic backup creation (`/opt/gtwy/backup-YYYYMMDD-HHMMSS/`)
  - Preserves `config.yml` (IONOS API keys, gateway settings)
  - Preserves `tunnels.db` (boxes and tunnels)
  - Database schema migration system
  - Version detection from installed binary or database
  - Validates configuration after update
  - `--force` flag to force update even if same version

- **Enhanced `add-box` output** - Copy-paste ready setup instructions
  - Displays complete `tnl setup` command with correct IP and ports
  - Clear step-by-step instructions for box setup
  - Formatted box with visual separators

#### tnl (Tunnel Client)
- **`update` command** - Update tnl to new version while preserving configuration
  - Automatic backup creation (`/etc/tnl/backup-YYYYMMDD-HHMMSS/`)
  - Preserves `config.yml` (gateway details, service tunnels)
  - Preserves SSH keys (`/home/tunneluser/.ssh/tunnel_key*`)
  - Graceful service restart (stops all tnl services, updates, restarts)
  - Config migration system for future upgrades
  - Real-time service status after restart
  - `--force` flag to force update even if same version

### Changed
- Project repository renamed to `tnld` (Tunnel Daemon Infrastructure)
  - Repository: https://github.com/steedalot/tnld
  - Tool names unchanged: `gtwy` and `tnl` remain the same

## [1.1.0] - 2025-12-14

### Added

#### gtwy (Gateway Manager)
- JSON response format for `request` command
  - Returns both `server_port` and `subdomain` as JSON
  - Enables tnl to display full URLs to users
  - More extensible for future enhancements

#### tnl (Tunnel Client)
- **`add <service> <port>` command** - Add service tunnel
  - Requests tunnel from gateway via SSH
  - Creates systemd service for persistent tunnel (tnl-<service>.service)
  - Displays full subdomain URL
  - Automatic rollback on failure (atomic operation)
  - One systemd service per tunnel for easy management

- **`remove <service>` command** - Remove service tunnel
  - Stops and disables systemd service
  - Releases tunnel on gateway (DNS, nginx, SSL cleanup)
  - Removes from local configuration
  - Clean teardown of all resources

- **`list` command** - List all service tunnels
  - Shows service name, local port, server port, URL, and status
  - Real-time systemd status check (active/inactive)
  - Formatted table output with totals

- **Central configuration file: `/etc/tnl/config.yml`**
  - Gateway connection details (IP, ports)
  - Service tunnel tracking
  - YAML format (human-readable, supports comments)
  - Permissions: root:root 644

### Changed
- `tnl setup` now creates `/etc/tnl/config.yml` with gateway connection details
- `gtwy request` output format: plain text → JSON (**BREAKING CHANGE**)
  - Old: `10001`
  - New: `{"server_port": 10001, "subdomain": "gitea.box01.kibox.online"}`
- Updated `tnl status` help text to clarify it shows admin tunnel status

## [1.0.0] - 2025-12-13

### Initial Release

#### gtwy (Gateway Manager)
- `install` command - One-time system setup
- `setup` command - Interactive configuration wizard
- `add-box` command - Register new IT.Box
- `remove-box` command - Remove box and all tunnels
- `list-boxes` command - Show registered boxes
- `get-port` command - Retrieve admin SSH port for box
- `request` command - Create service tunnel (called by box via SSH)
- `release` command - Remove service tunnel (called by box via SSH)
- `list` command - Show all active tunnels
- `info` command - Detailed tunnel information
- `version` command - Show version

#### tnl (Tunnel Client)
- `install` command - One-time client setup
- `setup` command - Configure admin SSH tunnel
- `status` command - Show tunnel status
- `version` command - Show version

### Security
- SSH key-based authentication only
- Command restriction in authorized_keys (no shell access)
- `restrict` flag prevents shell access, only port-forwarding allowed
- GatewayPorts disabled (admin tunnels only accessible from localhost)
- Minimal sudo permissions (only nginx reload and certbot)
- Separate port ranges for admin SSH and service tunnels

### Documentation
- Complete README for gtwy with command reference
- Complete README for tnl with installation guide
- TESTING.md with 4-phase test plan
- CLAUDE.md technical specification
- Architecture diagrams
- Troubleshooting guides

### Technical
- Single-file Python scripts
- Embedded nginx template in gtwy
- SQLite database for state management
- Python 3.8+ compatible
- Dependencies: PyYAML, requests (gtwy only)
- Idempotent installation

---

[2.0.0]: https://github.com/steedalot/itbox-tools/releases/tag/v2.0.0
[1.2.6]: https://github.com/steedalot/tnld/releases/tag/v1.2.6
[1.2.4]: https://github.com/steedalot/tnld/releases/tag/v1.2.4
[1.2.3]: https://github.com/steedalot/tnld/releases/tag/v1.2.3
[1.2.2]: https://github.com/steedalot/tnld/releases/tag/v1.2.2
[1.2.1]: https://github.com/steedalot/tnld/releases/tag/v1.2.1
[1.2.0]: https://github.com/steedalot/tnld/releases/tag/v1.2.0
[1.1.0]: https://github.com/steedalot/tnld/releases/tag/v1.1.0
[1.0.0]: https://github.com/steedalot/tnld/releases/tag/v1.0.0
