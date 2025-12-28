# IT.Box Tools

Collection of management tools for the IT.Box infrastructure - a decentralized educational platform for schools in Lower Saxony, Germany.

## 🛠️ Tools

### Infrastructure & Tunneling

- **[gtwy](gtwy/)** - Gateway server for SSH reverse tunnels with automated DNS, SSL, and nginx configuration
- **[tnl](tnl/)** - Client-side tunnel manager for IT.Boxes

### Container Management

- **[boxctl](boxctl/)** - Docker container management for IT.Box with automatic docker-compose generation
- **[catalogctl](catalogctl/)** - Container catalog management for publishing containers to IT.Boxes

### Authentication & Security

- **[mnchk](mnchk/)** - Credential checker for moin.schule SSO integration

## 📦 Quick Overview

| Tool | Purpose | Run On | Version |
|------|---------|--------|---------|
| gtwy | SSH tunnel gateway with DNS/SSL automation | Gateway Server | 2.0.0 |
| tnl | Tunnel client for IT.Boxes | IT.Box | 2.0.0 |
| boxctl | Container lifecycle management | IT.Box | 2.0.0 |
| catalogctl | Container catalog server | Catalog Server | 2.0.0 |
| mnchk | moin.schule credential validation | IT.Box | 2.0.0 |

## 🚀 Getting Started

Each tool has its own comprehensive documentation in its subdirectory. Click on the tool name above to access detailed installation instructions, usage examples, and API documentation.

### Typical Workflow

1. **Gateway Setup**: Install `gtwy` on your gateway server
2. **Box Setup**: Install `tnl` on each IT.Box
3. **Container Setup**: Install `boxctl` on IT.Boxes for container management
4. **Catalog Setup**: Install `catalogctl` on your catalog server (optional)
5. **Authentication**: Use `mnchk` for moin.schule SSO integration (optional)

## 📚 Documentation

- [gtwy Documentation](gtwy/README.md) - Gateway server management
- [tnl Documentation](tnl/README.md) - Tunnel client management
- [boxctl Documentation](boxctl/README.md) - Container management
- [catalogctl Documentation](catalogctl/README.md) - Catalog server management
- [mnchk Documentation](mnchk/README.md) - Credential checking

## 🏗️ Architecture

```
┌─────────────────────────────────────────────┐
│          IT.Box (Raspberry Pi 5)            │
│  ┌───────────────────────────────────────┐  │
│  │  boxctl - Container Management        │  │
│  │  - Gitea, Portainer, Open-WebUI       │  │
│  │  - Docker Compose automation          │  │
│  └───────────────────────────────────────┘  │
│  ┌───────────────────────────────────────┐  │
│  │  tnl - Tunnel Client                  │  │
│  │  - SSH reverse tunnels                │  │
│  │  - Service exposure                   │  │
│  └─────────────┬─────────────────────────┘  │
└────────────────┼────────────────────────────┘
                 │ SSH Tunnels
                 ▼
┌─────────────────────────────────────────────┐
│         Gateway Server (VPS/Cloud)          │
│  ┌───────────────────────────────────────┐  │
│  │  gtwy - Gateway Manager               │  │
│  │  - nginx reverse proxy                │  │
│  │  - Let's Encrypt SSL                  │  │
│  │  - IONOS DNS automation               │  │
│  └───────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
                 │
                 ▼
        🌍 Public Internet
  https://gitea.box01.kibox.online
  https://portainer.box01.kibox.online
```

## ⚙️ Requirements

### Common Requirements
- Python 3.8+
- Ubuntu/Debian Linux

### Tool-Specific Requirements
See individual tool READMEs for detailed requirements.

## 📝 Version History

All tools are currently at version **2.0.0** with the following unified features:

- ✅ Single-file tools (no external config files needed)
- ✅ Self-installing with `sudo <tool> install`
- ✅ Comprehensive error handling
- ✅ Detailed logging
- ✅ Zero-downtime updates
- ✅ HTTP Basic Authentication support (gtwy/tnl)
- ✅ Automated migrations

See individual tool CHANGELOGs for detailed version histories.

## 📄 License

MIT License

## 👨‍💻 Authors

Developed for the IT.Box / KI.Box infrastructure in Lower Saxony, Germany.

## 🔗 Links

- [IT.Box Project](https://github.com/n-21/itbox)
- [moin.schule Platform](https://moin.schule)

---

**IT.Box Tools** - Simple. Robust. Educational. 🚀
