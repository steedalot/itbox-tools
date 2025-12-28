# boxctl - IT.Box Container Management Tool

Version 0.1 - MVP

## Installation

```bash
# Abhängigkeiten installieren (einmalig)
sudo apt install python3-yaml python3-requests docker-compose

# boxctl ausführbar machen
chmod +x boxctl

# Optional: In /usr/local/bin verschieben
sudo mv boxctl /usr/local/bin/
```

## Erste Schritte

```bash
# 1. Installation (erstellt Verzeichnisstruktur)
sudo boxctl install

# 2. Container-Katalog synchronisieren
sudo boxctl sync

# 3. Verfügbare Container anzeigen
boxctl list

# 4. Container hinzufügen
sudo boxctl add portainer
sudo boxctl add mariadb
sudo boxctl add gitea

# 5. Status prüfen
boxctl status
```

## Commands

### `boxctl install`
Initialisiert boxctl und erstellt Verzeichnisstruktur:
- `/opt/itbox/config/boxctl/` - Katalog & State
- `/opt/itbox/compose/` - docker-compose.yml & .env
- `/opt/itbox/data/` - Container-Daten

**Benötigt sudo**

### `boxctl sync`
Lädt Container-Katalog vom Server:
- Katalog: `catalog.yaml`
- Container-Definitionen: `containers/*.yaml`

Cache wird lokal in `/opt/itbox/config/boxctl/` gespeichert.

**Benötigt sudo**

### `boxctl list`
Zeigt verfügbare Container gruppiert nach Kategorien:
- `[✓]` = installiert
- `[ ]` = verfügbar
- `(required)` / `(recommended)` = Tags

**Kein sudo nötig**

### `boxctl add <container>`
Installiert einen Container:
1. Prüft Dependencies
2. Prüft Port-Konflikte
3. Fragt Secrets ab (interaktiv)
4. Erstellt Verzeichnisse
5. Generiert docker-compose.yml
6. Updated .env
7. Startet Container
8. Zeigt Post-Install-Messages

**Benötigt sudo**

**Beispiel:**
```bash
sudo boxctl add luanti

Installing Luanti Game-Server
Description: Game-Server für Lua-Programmierung

Checking ports:
  ✓ 8400/udp available
  ✓ 8401/tcp available

Install container? [Y/n]: y

▸ Creating /opt/itbox/data/luanti... ✓
▸ Creating /opt/itbox/config/luanti/minetest.conf... ✓
▸ Updating docker-compose.yml... ✓
▸ Updating state... ✓
▸ Starting container... ✓

✅ Luanti Game-Server installed successfully!

  ✅ Luanti läuft auf Port 8400 (UDP)
  📱 Client herunterladen: https://www.luanti.org/downloads/
  🔗 Verbinden mit: <IT.Box-IP>:8400
```

### `boxctl remove <container>`
Entfernt einen Container:
- Stoppt Container
- Entfernt aus docker-compose.yml
- Fragt nach Daten-Löschung

**Benötigt sudo**

**Flags:**
- `--keep-data` - Behält Daten-Verzeichnis

**Beispiel:**
```bash
sudo boxctl remove luanti

Removing luanti
Are you sure? [y/N]: y

▸ Stopping container... ✓
▸ Updating docker-compose.yml... ✓
▸ Updating state... ✓

Remove data directory /opt/itbox/data/luanti? [y/N]: n

✅ luanti removed successfully!
```

### `boxctl status`
Zeigt Status aller installierten Container:
- Installierte Container mit Ports
- Docker-Status (docker-compose ps)

**Kein sudo nötig**

### `boxctl ports [container]`
Zeigt Port-Informationen:
- Ohne Argument: Alle Container
- Mit Argument: Spezifischer Container

Inkl. Tunnel-Hinweise aus Container-Definition.

**Kein sudo nötig**

**Beispiel:**
```bash
boxctl ports gitea

Gitea Ports

8201/tcp - HTTP-Port
  HTTP-Port - Für Tunnel: sudo tnl add gitea 8201
8202/tcp - SSH-Port für git clone
  SSH-Port für git clone via SSH
```

## Verzeichnisstruktur

```
/opt/itbox/
├── config/
│   └── boxctl/
│       ├── state.yaml              # Lokaler Zustand
│       ├── catalog.yaml            # Gecachter Katalog
│       └── containers/             # Container-Definitionen
│           ├── portainer.yaml
│           ├── gitea.yaml
│           └── ...
│
├── compose/
│   ├── docker-compose.yml          # Generiert von boxctl
│   └── .env                        # Secrets (generiert)
│
└── data/
    ├── portainer/
    ├── gitea/
    ├── luanti/
    └── ...
```

## Konfiguration

### Katalog-URL ändern

Standard: `https://raw.githubusercontent.com/n-21/itbox-containers/main`

Ändern in `/opt/itbox/config/boxctl/state.yaml`:
```yaml
catalog:
  url: "https://your-server.com/containers"
```

Dann: `sudo boxctl sync`

### Lokales Testen (ohne Server)

```bash
# Lokalen Katalog verwenden
export CATALOG_URL="file:///pfad/zu/container-definitions"

# Oder state.yaml manuell editieren
```

## Secrets

### Automatisch generiert
Container mit `generate: "password"` bekommen sichere Passwörter:
- 20 Zeichen
- Buchstaben, Zahlen, Sonderzeichen
- Automatisch in `.env` gespeichert

### Anzeigen
```bash
cat /opt/itbox/compose/.env
```

### Manuell ändern
```bash
sudo nano /opt/itbox/compose/.env
# Nach Änderung:
cd /opt/itbox/compose
sudo docker-compose restart <container>
```

## Troubleshooting

### Port bereits belegt
```
❌ Port 8400/udp already used by luanti
```
→ Port wird bereits verwendet, wähle anderen Container oder entferne existierenden

### Container startet nicht
```bash
# Logs anzeigen
cd /opt/itbox/compose
docker-compose logs <container>

# Container neu starten
docker-compose restart <container>
```

### Katalog-Download fehlgeschlagen
```bash
# Cache verwenden
ls /opt/itbox/config/boxctl/catalog.yaml

# Oder manuell herunterladen
curl -o /opt/itbox/config/boxctl/catalog.yaml \
  https://raw.githubusercontent.com/n-21/itbox-containers/main/catalog.yaml
```

### boxctl neu installieren
```bash
# Alte Installation entfernen
sudo rm -rf /opt/itbox/config/boxctl
sudo rm -rf /opt/itbox/compose

# Neu installieren
sudo boxctl install
sudo boxctl sync
```

## Entwicklung

### Neuen Container hinzufügen

1. Container-Definition erstellen:
   ```bash
   nano container-definitions/containers/mycontainer.yaml
   ```

2. In Katalog eintragen:
   ```bash
   nano container-definitions/catalog.yaml
   ```

3. Testen:
   ```bash
   sudo boxctl sync
   boxctl list
   sudo boxctl add mycontainer
   ```

### Container-Definition validieren

```bash
# YAML-Syntax prüfen
python3 -c "import yaml; yaml.safe_load(open('containers/mycontainer.yaml'))"

# Container installieren (Test)
sudo boxctl add mycontainer

# Bei Fehler: Logs prüfen
docker-compose logs mycontainer
```

## Bekannte Einschränkungen (v0.1)

- ❌ Keine `secrets set` Command (manuell in .env)
- ❌ Keine Container-Updates
- ❌ Keine Backup/Restore
- ❌ Keine Health-Checks
- ❌ Keine Multi-Box-Verwaltung

Geplant für v0.2+

## Version

```bash
boxctl version
# boxctl version 0.1.0
```

## Lizenz

MIT License - IT.Box Project
