# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2025-12-13

### Added

#### gtwy (Gateway Manager)
- `install` command - One-time system setup
  - Creates `/opt/gtwy` directory structure
  - Creates `tunneluser` and `gtwy-admin` group
  - Sets up SSH directory and permissions
  - Creates sudoers rules for nginx and certbot
  - Installs script globally to `/usr/local/bin/gtwy`
  - Embeds nginx template as constant (single-file tool)

- `setup` command - Interactive configuration wizard
  - IONOS DNS API configuration
  - Server IP and hostname detection
  - Domain whitelist configuration
  - Port range allocation (services and admin SSH)
  - Certbot email and staging mode
  - Creates `config.yml` and initializes SQLite database

- `add-box` command - Register new IT.Box
  - SSH public key validation and fingerprinting
  - Per-box domain assignment
  - Automatic admin SSH port allocation (20000-20999 range)
  - authorized_keys management with command restriction

- `remove-box` command - Remove box and all tunnels
  - Cascading tunnel deletion
  - DNS cleanup
  - Certificate revocation
  - authorized_keys cleanup

- `list-boxes` command - Show registered boxes
  - Display box ID, domain, SSH key info, admin port
  - Show tunnel count per box
  - Last seen timestamp

- `get-port` command - Retrieve admin SSH port for box

- `request` command - Create service tunnel (called by box via SSH)
  - Automatic server port allocation (10000-19999 range)
  - nginx configuration generation
  - DNS A record creation (IONOS API)
  - SSL certificate scheduling
  - Subdomain pattern: `<service>.<box-id>.<domain>`

- `release` command - Remove service tunnel (called by box via SSH)
  - nginx cleanup
  - DNS record deletion
  - Certificate revocation

- `list` command - Show all active tunnels
- `info` command - Detailed tunnel information
- `version` command - Show version

#### tnl (Tunnel Client)
- `install` command - One-time client setup
  - Creates `tunneluser` with home directory
  - Generates ED25519 SSH key pair
  - Installs autossh via apt
  - Copies script to `/usr/local/bin/tnl`
  - Displays public key for gateway registration
  - Handles existing users gracefully

- `setup` command - Configure admin SSH tunnel
  - Tests SSH connection to gateway
  - Creates systemd service for persistent tunnel
  - Configures autossh with proper reconnection settings
  - Reverse tunnel: `Gateway:{admin_port} → Box:22`
  - Auto-starts on boot
  - Supports custom SSH ports

- `status` command - Show tunnel status
  - systemd service status
  - Recent journal logs

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
- TESTING.md with 4-phase test plan (local, server setup, staging, production)
- CLAUDE.md technical specification
- Architecture diagrams
- Troubleshooting guides

### Technical
- Single-file Python scripts (gtwy ~2600 lines, tnl ~400 lines)
- Embedded nginx template in gtwy (no external files needed)
- SQLite database for state management
- Python 3.8+ compatible
- Dependencies: PyYAML, requests (gtwy only)
- Idempotent installation (can run install multiple times)

### Fixed
- Home directory creation when tunneluser exists without home
- SSH connection test now correctly handles command restriction
- Port confusion between admin_port (reverse tunnel) and ssh_port (connection)
- Proper separation of admin tunnels (static) and service tunnels (dynamic)

## [Unreleased]

### Planned for v1.1.0
- `tnl add <service> <port>` - Add service tunnel via admin tunnel
- `tnl remove <service>` - Remove service tunnel
- `tnl list` - List all service tunnels for this box
- Communication with gateway via admin SSH tunnel for tunnel requests

### Planned for v1.2.0
- `gtwy stats` - Gateway statistics
- `gtwy health` - System health check
- `gtwy test-tunnel` - End-to-end tunnel testing
- `gtwy sync-nginx` - Rebuild nginx from database
- `gtwy sync-dns` - Sync DNS records
- `gtwy sync-certs` - Request pending certificates
- `gtwy cleanup` - Remove orphaned resources

### Future Considerations
- Web dashboard
- Prometheus metrics export
- Automatic tunnel health monitoring
- Multi-gateway support (HA)
- Backup/restore functionality
- Rate limiting per box
- Webhook notifications

---

[1.0.0]: https://github.com/yourusername/tunnel-gateway/releases/tag/v1.0.0
