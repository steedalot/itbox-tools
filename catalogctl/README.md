# catalogctl - Container Catalog Management Tool

Version 0.1 - MVP

## Überblick

**catalogctl** verwaltet den Container-Katalog-Server für IT.Box-Installationen. Das Tool validiert und importiert Container-Definitionen (YAML-Dateien) in den zentralen Katalog, der dann von allen IT.Boxes über `boxctl sync` abgerufen werden kann.

## Installation

```bash
# Abhängigkeiten installieren (einmalig)
sudo apt install python3-yaml python3-requests nginx certbot python3-certbot-nginx

# catalogctl installieren (kopiert automatisch nach /usr/local/bin)
sudo ./catalogctl install
```

## Erste Schritte

```bash
# 1. Installation - Vollautomatisch!
#    - Installiert Script nach /usr/local/bin
#    - Richtet nginx ein
#    - Optional: SSL-Zertifikat via certbot
sudo ./catalogctl install

# 2. Container-Definition validieren
catalogctl validate portainer.yaml

# 3. Container importieren
sudo catalogctl import portainer.yaml

# 4. Katalog anzeigen
catalogctl list
```

## Commands

### `catalogctl install`

**Vollautomatische Installation** - Ein Befehl für alles!

Installiert catalogctl und richtet den kompletten Katalog-Server ein:
- Kopiert Script nach `/usr/local/bin/catalogctl` (falls nicht schon dort)
- Macht Script ausführbar
- Erstellt Verzeichnisstruktur:
  - `/opt/kibox/containers/` - Katalog-Root
  - `/opt/kibox/containers/containers/` - Container-Definitionen
  - `/opt/kibox/config/catalogctl/` - nginx-Config-Template
- Generiert und installiert nginx-Konfiguration
- Testet und lädt nginx neu
- Optional: Richtet SSL-Zertifikat via certbot ein

**Benötigt sudo**

**Beispiel:**
```bash
$ sudo ./catalogctl install

============================================================
  Installing catalogctl Script
============================================================

▸ Copying to /usr/local/bin/catalogctl... ✓
▸ Making executable... ✓

✅ catalogctl script installed to /usr/local/bin/

============================================================
  catalogctl Installation
============================================================

Container Catalog Configuration:
Domain [apps.kibox.online]:

▸ Creating directories... ✓
▸ Creating catalog.yaml... ✓
▸ Generating nginx config... ✓
▸ Setting permissions... ✓

▸ Installing nginx configuration... ✓
▸ Testing nginx configuration... ✓
▸ Reloading nginx... ✓

✅ catalogctl installed successfully!

SSL Certificate Setup:
Setup SSL certificate with certbot for apps.kibox.online? [Y/n]: y

▸ Running certbot for apps.kibox.online...

Saving debug log to /var/log/letsencrypt/letsencrypt.log
Requesting a certificate for apps.kibox.online
Successfully received certificate.
...

✅ SSL certificate installed for apps.kibox.online!

📦 Catalog available at: https://apps.kibox.online/containers

💡 Next steps:
  1. Import containers: sudo catalogctl import <file.yaml>
  2. View catalog: catalogctl list
```

### `catalogctl validate <file.yaml>`

Validiert Container-Definition ohne zu importieren:
- YAML-Syntax prüfen
- Schema validieren (Pflichtfelder)
- Port-Konflikte checken
- Dependencies prüfen
- Docker-Image validieren (optional)

**Kein sudo nötig**

**Beispiel:**
```bash
$ catalogctl validate luanti.yaml

============================================================
  Validating luanti.yaml
============================================================

▸ Loading YAML... ✓
▸ Validating schema... ✓
▸ Checking Docker image... ✓

Checking port conflicts:
  ✓ 8400/udp available
  ✓ 8401/tcp available

✅ Validation passed!
💡 Ready to import: sudo catalogctl import luanti.yaml
```

### `catalogctl import <file.yaml>`

Importiert Container-Definition in den Katalog:
1. Validiert YAML
2. Prüft Port-Konflikte
3. Prüft Dependencies
4. Kopiert nach `/opt/kibox/containers/containers/<id>.yaml`
5. Updated `catalog.yaml`
6. Setzt Permissions

**Benötigt sudo**

**Beispiel:**
```bash
$ sudo catalogctl import luanti.yaml

============================================================
  Importing luanti
============================================================

▸ Validating... ✓
▸ Checking port conflicts... ✓
▸ Copying to /opt/kibox/containers/containers/luanti.yaml... ✓
▸ Updating catalog.yaml... ✓
▸ Setting permissions... ✓

✅ luanti imported successfully!

💡 Container: Luanti Game-Server
💡 Category: games
💡 Version: 1.0

📦 View catalog: catalogctl list
```

### `catalogctl remove <container>`

Entfernt Container aus dem Katalog:
- Entfernt aus `catalog.yaml`
- Optional: Löscht YAML-Datei

**Benötigt sudo**

**Flags:**
- `--keep-file` - Behält YAML-Datei

**Beispiel:**
```bash
$ sudo catalogctl remove luanti

============================================================
  Removing luanti
============================================================

Are you sure? [y/N]: y

▸ Updating catalog.yaml... ✓

Delete /opt/kibox/containers/containers/luanti.yaml? [y/N]: y

▸ Deleting /opt/kibox/containers/containers/luanti.yaml... ✓

✅ luanti removed from catalog!
```

### `catalogctl list`

Zeigt alle Container im Katalog:
- Gruppiert nach Kategorien
- Mit Version und Beschreibung
- Zeigt Update-Zeitstempel

**Kein sudo nötig**

**Beispiel:**
```bash
$ catalogctl list

============================================================
  Container Catalog
============================================================

🏗️ Infrastruktur:
  portainer            v1.0   Docker Container Management UI
  mariadb              v1.0   MySQL-kompatible Datenbank

💻 Entwicklung:
  gitea                v1.0   Git-Repository-Server
  code-server          v1.0   VS Code im Browser

📚 Pädagogische Apps:
  langflow             v1.0   Visueller LLM Flow Builder

🎮 Game-Server:
  luanti               v1.0   Minecraft-ähnlicher Game-Server

Total: 6 containers
Updated: 2025-12-28T15:30:00Z
```

### `catalogctl export <container>`

Exportiert Container-Definition als YAML (stdout):
- Zum Bearbeiten
- Zum Backup
- Zum Teilen

**Kein sudo nötig**

**Beispiel:**
```bash
$ catalogctl export luanti > luanti-backup.yaml

$ catalogctl export portainer
id: portainer
name: "Portainer"
version: "1.0"
description: "Docker Container Management UI"
...
```

## Verzeichnisstruktur

```
/opt/kibox/
├── containers/                      # Katalog-Root (nginx-Root)
│   ├── catalog.yaml                 # Haupt-Index
│   ├── containers/                  # Container-Definitionen
│   │   ├── portainer.yaml
│   │   ├── mariadb.yaml
│   │   ├── gitea.yaml
│   │   └── ...
│   └── README.md                    # Katalog-Dokumentation (optional)
│
└── config/
    └── catalogctl/
        └── apps.kibox.online.conf   # nginx-Config-Template
```

## Workflow: Container hinzufügen

### 1. Container-YAML erstellen (mit Claude)

```bash
# Im Chat mit Claude
User: "Erstelle eine Container-Definition für Moodle LMS"

# Claude generiert komplette YAML-Datei
# User speichert als moodle.yaml
```

### 2. Validieren

```bash
catalogctl validate moodle.yaml

# Prüft:
# - YAML-Syntax
# - Pflichtfelder
# - Port-Konflikte
# - Dependencies
```

### 3. Optional: Bearbeiten

```bash
nano moodle.yaml

# Anpassungen vornehmen:
# - Ports ändern
# - Secrets hinzufügen
# - Post-Install-Messages
```

### 4. Importieren

```bash
sudo catalogctl import moodle.yaml

# Fügt zum Katalog hinzu
# Aktualisiert catalog.yaml
```

### 5. Veröffentlichen

```bash
# Katalog ist sofort verfügbar unter:
https://apps.kibox.online/containers/

# IT.Boxes können abrufen:
sudo boxctl sync
```

## Port-Management

### Port-Ranges (Best Practices)

| Range      | Verwendung              | Beispiele         |
|------------|-------------------------|-------------------|
| 9000-9099  | Management/Monitoring   | Portainer: 9000   |
| 8100-8199  | Datenbanken             | MariaDB: 8106     |
| 8200-8299  | Entwicklung             | Gitea: 8201-8202  |
| 8300-8399  | Pädagogische Apps       | Langflow: 8320    |
| 8400-8499  | Game-Server             | Luanti: 8400-8401 |

### Port-Konflikte prüfen

```bash
# Vor Import: Validierung prüft automatisch
catalogctl validate myapp.yaml

# Manuell prüfen
catalogctl list | grep "Port"
```

## Kategorie-System

Vordefinierte Kategorien in `catalog.yaml`:

- `infrastructure` - Infrastruktur (Portainer, Datenbanken)
- `development` - Entwicklung (Git, IDEs, CI/CD)
- `education` - Pädagogische Apps (LMS, KI-Tools)
- `games` - Game-Server (Luanti, Minecraft, etc.)

Neue Kategorien können in `catalog.yaml` hinzugefügt werden.

## nginx-Konfiguration

Die generierte nginx-Config bietet:

- **Static File Serving**: Liefert YAML-Dateien aus
- **CORS-Header**: Erlaubt Zugriff von allen IT.Boxes
- **Directory Listing**: Katalog-Browser (optional)
- **MIME-Types**: Korrekte Content-Types für YAML/MD
- **Health-Check**: `/health` Endpoint
- **SSL-Ready**: certbot-kompatibel

## Troubleshooting

### Port-Konflikt

```
❌ Port 8400/udp already used by luanti
```

**Lösung:** Port in YAML-Datei ändern und erneut validieren

### Missing Dependencies

```
⚠️  Missing dependencies:
  - mariadb
```

**Lösung:**
1. Dependency zuerst importieren: `sudo catalogctl import mariadb.yaml`
2. Oder: Ignorieren und trotzdem importieren (bei `import` gefragt)

### YAML-Syntax-Fehler

```
❌ Invalid YAML syntax
```

**Lösung:** YAML-Datei mit Validator prüfen:
```bash
python3 -c "import yaml; yaml.safe_load(open('file.yaml'))"
```

### Container nicht gefunden

```
❌ Container 'xyz' not found in catalog
```

**Lösung:** Mit `catalogctl list` prüfen, welche Container verfügbar sind

## Sicherheit

### Permissions

- Katalog-Verzeichnis: `www-data:www-data`
- YAML-Dateien: `644` (lesbar für nginx)
- nginx-Config: `root:root`, `644`

### CORS-Policy

Default: Alle Origins erlaubt (`*`)

Für Production: In nginx-Config einschränken:
```nginx
add_header 'Access-Control-Allow-Origin' 'https://trusted-domain.com' always;
```

### SSL/TLS

**Pflicht für Production!**

```bash
sudo certbot --nginx -d apps.kibox.online
```

Auto-Renewal:
```bash
sudo certbot renew --dry-run
```

## Integration mit boxctl

### IT.Box-Seite

```bash
# Katalog-URL konfigurieren (bei boxctl install)
Catalog URL: https://apps.kibox.online/containers

# Katalog synchronisieren
sudo boxctl sync

# Container installieren
sudo boxctl add portainer
```

### Server-Seite

```bash
# Container zum Katalog hinzufügen
sudo catalogctl import portainer.yaml

# Sofort verfügbar für alle IT.Boxes
```

## Backup & Restore

### Backup

```bash
# Kompletten Katalog sichern
tar -czf catalog-backup-$(date +%Y%m%d).tar.gz /opt/kibox/containers/

# Nur YAML-Dateien
cp -r /opt/kibox/containers/containers /backup/
```

### Restore

```bash
# Kompletten Katalog wiederherstellen
tar -xzf catalog-backup-20251228.tar.gz -C /

# Permissions wiederherstellen
sudo chown -R www-data:www-data /opt/kibox/containers/
```

## Entwicklung

### Neue Container-Definition erstellen

1. **Mit Claude:**
   ```
   User: "Erstelle Container-Definition für XYZ"
   Claude: Generiert komplette YAML basierend auf CLAUDE-CONTAINER-YAML-GUIDE.md
   ```

2. **Validieren:**
   ```bash
   catalogctl validate xyz.yaml
   ```

3. **Importieren:**
   ```bash
   sudo catalogctl import xyz.yaml
   ```

### Container-Definition bearbeiten

```bash
# Exportieren
catalogctl export xyz > xyz.yaml

# Bearbeiten
nano xyz.yaml

# Re-importieren (überschreibt)
sudo catalogctl import xyz.yaml
```

### Katalog versionieren (Git)

```bash
cd /opt/kibox/containers

# Git initialisieren
git init
git add catalog.yaml containers/

# Nach jedem Import committen
git commit -m "Add container: xyz"

# Optional: Auf Remote pushen
git remote add origin https://github.com/your-org/itbox-catalog
git push origin main
```

## Bekannte Einschränkungen (v0.1)

- ❌ Keine Docker-Hub-Validierung (Image-Existenz wird nicht geprüft)
- ❌ Keine automatische Versionierung
- ❌ Keine Dependency-Auto-Installation
- ❌ Keine Web-UI
- ❌ Keine Download-Statistiken

Geplant für v0.2+

## Version

```bash
catalogctl version
# catalogctl version 0.1.0
```

## Lizenz

MIT License - IT.Box Project

## Support

- **Issues:** GitHub Issues
- **Dokumentation:** [boxctl-container-format.md](Konzept/boxctl-container-format.md)
- **Beispiele:** Siehe existierender Katalog unter https://apps.kibox.online/containers
