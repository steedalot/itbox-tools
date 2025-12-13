# Tunnel Gateway Manager

Self-hosted SSH reverse tunnel management system for exposing services behind NAT/firewall.

## Overview

This project provides two complementary tools for managing SSH reverse tunnels:

- **gtwy** - Server-side gateway manager with automated DNS, SSL, and nginx configuration
- **tnl** - Client-side tunnel manager for IT.Boxes

Perfect for self-hosted infrastructure where services need to be exposed through a central gateway server.

## Features

### gtwy (Gateway Server)

- 🔧 **Automated Setup** - One-command installation and configuration
- 🌐 **DNS Automation** - IONOS DNS API integration
- 🔒 **SSL Certificates** - Automatic Let's Encrypt certificate management
- 🔄 **nginx Integration** - Dynamic reverse proxy configuration
- 📊 **Multi-Domain** - Support for multiple domains and subdomains
- 🗄️ **SQLite Database** - Built-in state management
- 🔑 **SSH-based Auth** - Secure key-based authentication

### tnl (Tunnel Client)

- 🚀 **Easy Installation** - Automated user and key setup
- 🔄 **Persistent Tunnels** - systemd + autossh for reliability
- 📡 **Admin Tunnel** - Reverse SSH access for management
- ⚡ **Auto-Reconnect** - Automatic recovery from network issues

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                   IT.Box (Client)                   │
│  ┌──────────────────────────────────────────────┐  │
│  │  Services: Gitea, Theia, Portainer, etc.    │  │
│  │  Ports: 3000, 8080, 9000, ...                │  │
│  │  tnl - Tunnel Client                         │  │
│  └────────────────┬─────────────────────────────┘  │
│                   │ SSH Reverse Tunnel             │
└───────────────────┼─────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────┐
│              Gateway Server (gtwy)                  │
│  ┌──────────────────────────────────────────────┐  │
│  │  gtwy - Gateway Manager                      │  │
│  │  - SSH tunnels (ports 10000-19999)           │  │
│  │  - nginx reverse proxy                       │  │
│  │  - Let's Encrypt SSL                         │  │
│  │  - IONOS DNS automation                      │  │
│  └────────────────┬─────────────────────────────┘  │
└───────────────────┼─────────────────────────────────┘
                    │
                    ▼
              🌍 Internet
    https://gitea.box01.kibox.online
    https://theia.box01.kibox.online
    https://portainer.box01.kibox.online
```

## Quick Start

### Server Setup (Gateway)

```bash
# Download gtwy
curl -O https://github.com/yourusername/tunnel-gateway/releases/latest/download/gtwy
chmod +x gtwy

# Install (creates users, permissions, etc.)
sudo ./gtwy install

# Configure (interactive wizard)
sudo gtwy setup

# Add a box
sudo gtwy add-box box01 kibox.online '<ssh-public-key>'
```

### Client Setup (IT.Box)

```bash
# Download tnl
curl -O https://github.com/yourusername/tunnel-gateway/releases/latest/download/tnl
chmod +x tnl

# Install (creates user, generates SSH key)
sudo ./tnl install
# Copy the displayed public key and register on gateway

# Setup tunnel (get admin_port from: gtwy get-port box01)
sudo tnl setup gateway.example.com 20001
```

**Done!** The admin tunnel is now running. From the gateway server:

```bash
ssh -p 20001 user@localhost  # SSH to the box
```

## Documentation

- [gtwy Documentation](gtwy/README.md) - Server-side gateway manager
- [tnl Documentation](tnl/README.md) - Client-side tunnel manager
- [Testing Guide](docs/TESTING.md) - Complete testing workflow
- [Technical Specification](docs/CLAUDE.md) - Architecture and implementation details

## Requirements

### Gateway Server

- Ubuntu/Debian Linux
- Python 3.8+
- nginx
- certbot (Let's Encrypt)
- IONOS account with DNS API access

### IT.Box (Client)

- Ubuntu/Debian Linux
- Python 3.8+
- autossh
- OpenSSH client

## Installation

See the Quick Start section above, or refer to:
- [gtwy/README.md](gtwy/README.md) for detailed gateway setup
- [tnl/README.md](tnl/README.md) for detailed client setup
- [docs/TESTING.md](docs/TESTING.md) for complete testing guide

## Use Cases

- **Self-hosted Services** - Expose Gitea, Nextcloud, etc. from behind NAT
- **IoT Devices** - Manage devices without public IPs
- **Remote Development** - Access code-server/Theia instances
- **Multi-tenant Hosting** - Separate domains per box
- **Educational Institutions** - School IT.Boxes with centralized gateway

## Security

- SSH key-based authentication only
- Command restriction in authorized_keys (no shell access)
- Automatic SSL/TLS via Let's Encrypt
- GatewayPorts disabled (tunnels only accessible from gateway)
- Minimal sudo permissions for tunnel operations
- Separation of admin tunnels (SSH) and service tunnels (HTTP/HTTPS)

## Versioning

This project uses [Semantic Versioning](https://semver.org/):

- **v1.0.0** - Initial release (current)
  - gtwy: install, setup, add-box, list-boxes, get-port, request, release
  - tnl: install, setup, status

- **v1.1.0** - Service tunnels (planned)
  - tnl: add, remove, list commands
  - Dynamic service tunnel management

## License

MIT License - See [LICENSE](LICENSE)

## Contributing

This is currently a private project for IT.Box infrastructure. Contributions, bug reports, and feature requests are welcome via GitHub issues.

## Support

- **Documentation**: See [docs/](docs/) directory
- **Issues**: GitHub Issues
- **Testing**: See [docs/TESTING.md](docs/TESTING.md)

## Authors

Developed for the KI.Box / IT.Box infrastructure.

---

**tunnel-gateway** - Simple. Robust. Self-hosted. 🚀
