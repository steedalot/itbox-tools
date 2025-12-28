# Anleitung: Container-YAML-Definitionen erstellen

**Für:** Claude (AI Assistant)
**Zweck:** Container-Definitionen für IT.Box/KI.Box-Katalog generieren
**Format-Version:** 1.0
**Letzte Aktualisierung:** 2025-12-28

---

## Kontext: Worum geht es?

### IT.Box - Dezentrale Bildungsinfrastruktur

**IT.Box** ist eine dezentrale Bildungsinfrastruktur für Schulen in Niedersachsen:
- **Hardware:** Raspberry Pi 5 (16GB) in jeder Schule (~300€)
- **Software:** Docker-basierte Container für Bildungsanwendungen
- **Ziel:** Schulen können eigene Apps entwickeln und teilen, KI nutzen, bei voller Datensouveränität

### Das Ökosystem

```
┌────────────────────────────────────────────────────────────┐
│ Katalog-Server (apps.kibox.online)                         │
│ - Zentrale Container-Definitionen (YAML-Dateien)           │
│ - Verwaltet mit: catalogctl                                │
│ - Du (Claude) generierst die YAMLs                         │
└────────────────────────────────────────────────────────────┘
                          ↓
                   (HTTPS Download)
                          ↓
┌────────────────────────────────────────────────────────────┐
│ IT.Box in Schule (Raspberry Pi)                            │
│ - Lädt Katalog: boxctl sync                                │
│ - Installiert Container: boxctl add portainer              │
│ - Läuft lokal in der Schule                                │
└────────────────────────────────────────────────────────────┘
```

### Deine Rolle

1. **User fragt:** "Erstelle Container-Definition für Moodle LMS"
2. **Du prüfst:** Live-Katalog unter https://apps.kibox.online/containers/
3. **Du generierst:** Vollständige YAML-Datei (siehe unten)
4. **User importiert:** `catalogctl import moodle.yaml`
5. **Schulen nutzen:** `boxctl add moodle` auf ihren IT.Boxes

### Warum YAML-Definitionen?

Jeder Container braucht:
- Docker-Image + Version
- Ports (ohne Konflikte!)
- Volumes (Datenspeicherung)
- Secrets (Passwörter, API-Keys)
- Dependencies (z.B. Datenbank)
- Post-Install-Messages (Was muss User wissen?)

**Deine Aufgabe:** Vollständige, konfliktfreie, funktionierende YAML-Definitionen generieren.

---

## Workflow

### 1. Katalog prüfen (WICHTIG!)

**Bevor** du eine Container-Definition erstellst, **IMMER** den aktuellen Katalog abrufen:

```
Katalog-URL: https://apps.kibox.online/containers/
```

**Prüfe:**
- Existierende Container (Port-Ranges, Dependencies)
- Verfügbare Kategorien
- Port-Konflikte vermeiden
- Dependencies korrekt referenzieren

**Beispiel-Abruf:**
```bash
# catalog.yaml abrufen
curl https://apps.kibox.online/containers/catalog.yaml

# Einzelne Container-Definition ansehen
curl https://apps.kibox.online/containers/containers/portainer.yaml
```

### 2. Container-Definition generieren

Basierend auf User-Request und Katalog-Status:
- Vollständige YAML-Datei erstellen
- Alle Pflichtfelder ausfüllen
- Port-Konflikte vermeiden (Katalog checken!)
- Dependencies korrekt setzen
- Post-Install-Messages schreiben

### 3. Validierung empfehlen

User sollte vor Import validieren:
```bash
catalogctl validate <container>.yaml
```

---

## YAML-Format-Spezifikation

### Pflichtfelder

```yaml
id: string                    # Eindeutig, lowercase, a-z0-9-
name: string                  # Anzeigename
version: string               # SemVer (z.B. "1.0")
description: string           # Kurzbeschreibung (1 Zeile)
category: string              # infrastructure|development|education|games

image:
  name: string                # Docker-Image (z.B. "portainer/portainer-ce")
  tag: string                 # Tag (meist "latest")

ports:                        # Array, mindestens 1 Port
  - host: int                 # Port auf IT.Box (8100-8499)
    container: int            # Port im Container
    protocol: string          # "tcp" oder "udp"
    description: string       # Wofür ist der Port?
    bind: string              # Optional: "127.0.0.1" (nur localhost) oder "0.0.0.0" (alle IPs)

volumes:                      # Array, mindestens 1 Volume
  - host: string              # Pfad (meist /opt/itbox/data/<id>/)
    container: string         # Mount-Point im Container
    description: string       # Was wird gespeichert?
    readonly: boolean         # Optional: true für Read-Only Mount
```

### Optionale Felder

```yaml
links:                        # URLs für Dokumentation
  homepage: string
  docs: string
  download: string
  github: string              # Optional: GitHub Repository URL

global_env:                   # Globale IT.Box Variablen (in .env)
  - key: string               # Name der globalen Variable
    description: string       # Wofür wird sie genutzt?
    required: boolean         # Muss bei boxctl add existieren?
    default: string           # Optional: Default-Wert falls nicht gesetzt

environment:                  # Statische ENV-Variablen
  - key: string
    value: string             # Kann ${SECRET_NAME} oder ${GLOBAL_VAR} referenzieren

secrets:                      # Dynamische Secrets (in .env)
  - key: string               # ENV-Variable-Name
    description: string       # Was ist das?
    required: boolean         # Beim Install abfragen?
    generate: string          # "password"|"secret"|"api_key"

prompts:                      # Interaktive Konfiguration bei Installation
  - key: string               # ENV-Variable-Name
    question: string          # Frage an User
    default: string           # Optional: Default-Wert
    description: string       # Optional: Erklärung/Hinweis
    options: []               # Optional: Multiple-Choice Liste (Array von Strings)
    validate: string          # Optional: "int"|"float"|"url"

depends_on:                   # Array von Container-IDs
  - string

configs:                      # Auto-generierte Config-Dateien
  - path: string              # Absoluter Pfad
    description: string
    content: |                # Multiline-String
      <config file content>

post_install:
  messages:                   # Array von Strings
    - string                  # Was nach Installation anzeigen?

  ports_info:                 # Tunnel-Hinweise
    - port: int
      protocol: string
      description: string     # z.B. "Für Tunnel: sudo tnl add..."

resources:                    # Geschätzte Ressourcen
  ram_mb: int
  cpu_cores: float
  disk_mb: int

# Nur in catalog.yaml:
required: boolean             # Container kann nicht entfernt werden
recommended: boolean          # Bei Setup vorgeschlagen
warnings:                     # Array von Strings
  - string
tags:                         # Such-Tags
  - string
```

---

## Port-Ranges (WICHTIG!)

**Immer Katalog checken, um Konflikte zu vermeiden!**

| Range      | Kategorie           | Beispiele             |
|------------|---------------------|-----------------------|
| 9000-9099  | Management          | Portainer: 9000       |
| 8100-8199  | Datenbanken         | MariaDB: 8106         |
| 8200-8299  | Entwicklung         | Gitea: 8201-8202      |
|            |                     | Code-Server: 8200     |
| 8300-8399  | Pädagogische Apps   | Langflow: 8320        |
|            |                     | Open-WebUI: 8310      |
| 8400-8499  | Game-Server         | Luanti: 8400-8401     |
|            |                     | Minecraft: 8450       |

**Best Practices:**
- Ports sequentiell vergeben (8301, 8302, ...)
- Katalog vorher abrufen!
- Bei Multi-Port: Zusammenhängend (8201, 8202)
- UDP nur wenn nötig (Game-Server)
- Privilegierte Ports (< 1024) vermeiden

---

## Kategorien

Verfügbar (Stand: catalog.yaml):
- `infrastructure` - Infrastruktur (🏗️)
- `development` - Entwicklung (💻)
- `education` - Pädagogische Apps (📚)
- `games` - Game-Server (🎮)

Wenn neue Kategorie nötig: User informieren, dass diese in catalog.yaml hinzugefügt werden muss.

---

## Beispiele

### Minimal-Beispiel (ohne Secrets)

```yaml
id: luanti
name: "Luanti Game-Server"
version: "1.0"
description: "Minecraft-ähnlicher Game-Server mit Lua-Modding"
category: games

links:
  homepage: "https://www.luanti.org/"
  docs: "https://wiki.luanti.org/"

image:
  name: "ghcr.io/luanti-org/luanti"
  tag: "latest"

ports:
  - host: 8400
    container: 30000
    protocol: udp
    description: "Spiel-Port"

volumes:
  - host: "/opt/itbox/data/luanti"
    container: "/var/lib/minetest"
    description: "Welten, Mods, Spieler-Daten"

environment: []
secrets: []
depends_on: []

post_install:
  messages:
    - "✅ Luanti läuft auf Port 8400 (UDP)"
    - "📱 Client: https://www.luanti.org/downloads/"
    - "🔗 Verbinden: <itbox-ip>:8400"

resources:
  ram_mb: 500
  cpu_cores: 0.5
  disk_mb: 100
```

### Mit Secrets & Dependencies

```yaml
id: gitea
name: "Gitea Git-Server"
version: "1.0"
description: "Git-Repository-Server mit Web-UI"
category: development

links:
  homepage: "https://gitea.io/"
  docs: "https://docs.gitea.io/"

image:
  name: "gitea/gitea"
  tag: "latest"

ports:
  - host: 8201
    container: 3000
    protocol: tcp
    description: "HTTP-Port"

  - host: 8202
    container: 22
    protocol: tcp
    description: "SSH-Port für git clone"

volumes:
  - host: "/opt/itbox/data/gitea"
    container: "/data"
    description: "Git-Repositories und Datenbank"

environment:
  - key: "USER_UID"
    value: "1000"

  - key: "GITEA__database__DB_TYPE"
    value: "mysql"

  - key: "GITEA__database__HOST"
    value: "mariadb:3306"

  - key: "GITEA__database__NAME"
    value: "gitea"

  - key: "GITEA__database__USER"
    value: "itbox"

  - key: "GITEA__database__PASSWD"
    value: "${GITEA_DATABASE_PASSWORD}"

secrets:
  - key: "GITEA_DATABASE_PASSWORD"
    description: "MariaDB-Passwort für Gitea"
    required: true
    generate: "password"

depends_on:
  - mariadb

post_install:
  messages:
    - "✅ Gitea läuft auf http://itbox.local:8201"
    - "🔧 Ersteinrichtung im Browser durchführen"
    - "📝 Datenbank ist bereits vorkonfiguriert"

  ports_info:
    - port: 8201
      protocol: tcp
      description: "HTTP-Port - Für Tunnel: sudo tnl add gitea 8201"

resources:
  ram_mb: 300
  cpu_cores: 0.3
  disk_mb: 50
```

### Mit optionalen API-Keys

```yaml
id: langflow
name: "Langflow"
version: "1.0"
description: "Visueller LLM Flow Builder"
category: education

links:
  homepage: "https://www.langflow.org/"
  docs: "https://docs.langflow.org/"

image:
  name: "langflowai/langflow"
  tag: "latest"

ports:
  - host: 8320
    container: 7860
    protocol: tcp
    description: "Web-UI"

volumes:
  - host: "/opt/itbox/data/langflow"
    container: "/app/langflow"
    description: "Flows und Datenbank"

environment:
  - key: "LANGFLOW_AUTO_LOGIN"
    value: "true"

  - key: "LANGFLOW_SUPERUSER"
    value: "admin"

  - key: "LANGFLOW_SUPERUSER_PASSWORD"
    value: "${LANGFLOW_SUPERUSER_PASSWORD}"

  - key: "LANGFLOW_SECRET_KEY"
    value: "${LANGFLOW_SECRET_KEY}"

secrets:
  - key: "LANGFLOW_SUPERUSER_PASSWORD"
    description: "Admin-Passwort für Langflow"
    required: true
    generate: "password"

  - key: "LANGFLOW_SECRET_KEY"
    description: "Secret Key für Session-Verwaltung"
    required: true
    generate: "secret"

optional_secrets:
  - key: "OPENAI_API_KEY"
    description: "OpenAI API Key für GPT-Modelle"
    docs: "https://platform.openai.com/api-keys"

  - key: "ANTHROPIC_API_KEY"
    description: "Anthropic API Key für Claude"
    docs: "https://console.anthropic.com/"

depends_on: []

post_install:
  messages:
    - "✅ Langflow läuft auf http://itbox.local:8320"
    - "👤 Login: admin / <generiertes Passwort>"
    - "🔑 API-Keys später hinzufügen: sudo boxctl secrets set OPENAI_API_KEY"

resources:
  ram_mb: 400
  cpu_cores: 0.5
  disk_mb: 100
```

---

## Best Practices

### 1. Katalog-Check ist Pflicht

```yaml
# ❌ FALSCH: Ohne Katalog-Check
User: "Erstelle Container für Moodle"
Claude: [Generiert YAML mit zufälligen Ports]

# ✅ RICHTIG: Mit Katalog-Check
User: "Erstelle Container für Moodle"
Claude:
  1. Ruft https://apps.kibox.online/containers/catalog.yaml ab
  2. Prüft belegte Ports
  3. Prüft Dependencies (z.B. mariadb vorhanden?)
  4. Generiert YAML mit freien Ports
  5. Warnt bei fehlenden Dependencies
```

### 2. Volumes

```yaml
# ✅ Daten in /opt/itbox/data/<container>/
volumes:
  - host: "/opt/itbox/data/moodle"
    container: "/var/www/html"
    description: "Moodle-Installation und Uploads"

# ✅ Configs in /opt/itbox/config/<container>/
  - host: "/opt/itbox/config/moodle"
    container: "/etc/moodle"
    description: "Konfigurationsdateien"

# ❌ Nicht: Home-Verzeichnisse, System-Pfade
  - host: "/home/user/data"  # FALSCH
  - host: "/etc"              # FALSCH
```

### 3. Secrets

```yaml
# ✅ Auto-Generate für DB-Passwörter
secrets:
  - key: "DB_PASSWORD"
    generate: "password"
    required: true

# ✅ Optional für API-Keys
optional_secrets:
  - key: "OPENAI_API_KEY"
    docs: "https://platform.openai.com/api-keys"

# ❌ Nicht: Passwörter hardcoden
environment:
  - key: "ADMIN_PASSWORD"
    value: "admin123"  # FALSCH!
```

### 4. Post-Install-Messages

```yaml
# ✅ Kurz, konkret, hilfreich
post_install:
  messages:
    - "✅ Service läuft auf http://itbox.local:8300"
    - "👤 Login: admin / <generiertes Passwort>"
    - "🔧 Ersteinrichtung erforderlich"

# ❌ Nicht: Zu ausführlich, redundant
post_install:
  messages:
    - "Der Container wurde erfolgreich installiert und läuft jetzt."
    - "Sie können jetzt über Ihren Browser auf den Service zugreifen."
    - "Weitere Informationen finden Sie in der Dokumentation."
    # Zu viel Text!
```

### 5. Dependencies

```yaml
# ✅ Katalog prüfen, ob Dependency existiert
depends_on:
  - mariadb  # Existiert im Katalog?

# ✅ Warnung generieren, wenn nicht im Katalog
# In Antwort an User:
"⚠️  Hinweis: Diese Container-Definition benötigt 'mariadb'.
Bitte stelle sicher, dass mariadb im Katalog verfügbar ist,
oder importiere es zuerst."

# ❌ Nicht: Nicht-existierende Dependencies
depends_on:
  - postgresql  # Gibt's nicht im Katalog!
```

### 6. LLM-Proxy Integration

IT.Box stellt einen zentralen LLM-Proxy bereit: `https://llm.kibox.online/v1` (LiteLLM)

**Standard-Integration für LLM-Container:**

```yaml
# ✅ Globale Variablen definieren
global_env:
  - key: "ITBOX_LLM_ENDPOINT"
    description: "IT.Box LLM-Proxy Endpoint (LiteLLM)"
    required: true
    default: "https://llm.kibox.online/v1"

  - key: "ITBOX_LLM_API_KEY"
    description: "IT.Box LLM API Key (von n-21 erhalten)"
    required: true

# ✅ In Container-ENV mappen (OpenAI-kompatibel)
environment:
  - key: "OPENAI_API_BASE"
    value: "${ITBOX_LLM_ENDPOINT}"

  - key: "OPENAI_API_KEY"
    value: "${ITBOX_LLM_API_KEY}"

# ✅ Optionale externe API-Keys
optional_secrets:
  - key: "CUSTOM_ANTHROPIC_API_KEY"
    description: "Eigener Anthropic Key (zusätzlich zu IT.Box LLM)"
    docs: "https://console.anthropic.com/"
```

**Best Practices:**
- Immer IT.Box LLM-Proxy als Standard
- Externe API-Keys nur als `optional_secrets`
- Keine vLLM-spezifischen Keys (IT.Box nutzt zentralen Proxy)
- OpenAI-kompatible ENV-Namen bevorzugen

**Workflow:**
1. `boxctl install` fragt nach `ITBOX_LLM_API_KEY`
2. Key wird in `/opt/itbox/compose/.env` gespeichert (global)
3. Alle LLM-Container nutzen automatisch den Proxy
4. User kann später eigene externe Keys setzen

### 7. Interaktive Prompts

Für Container die bei Installation konfiguriert werden müssen, nutze `prompts`:

**Einfaches Text-Prompt:**
```yaml
prompts:
  - key: "SERVER_NAME"
    question: "Server-Name?"
    default: "IT.Box Server"
    description: "Wird im Server-Browser angezeigt"
```

**Multiple-Choice:**
```yaml
prompts:
  - key: "GAME_MODE"
    question: "Welcher Spielmodus?"
    options:
      - "survival"
      - "creative"
      - "adventure"
      - "spectator"
    default: "creative"
    description: "Creative empfohlen für Schulen"
```

**Mit Validierung:**
```yaml
prompts:
  - key: "MAX_PLAYERS"
    question: "Maximale Spieleranzahl?"
    default: "20"
    validate: "int"
    description: "Empfohlen: 10-30 für Raspberry Pi"

  - key: "GPU_MEMORY"
    question: "GPU Memory Utilization (0.0-1.0)?"
    default: "0.9"
    validate: "float"
```

**In environment referenzieren:**
```yaml
prompts:
  - key: "VLLM_MODEL"
    question: "Welches LLM-Modell?"
    options:
      - "meta-llama/Llama-3.1-8B"
      - "mistralai/Mistral-7B-v0.1"
    default: "meta-llama/Llama-3.1-8B"

environment:
  - key: "MODEL_NAME"
    value: "${VLLM_MODEL}"  # Nutzt User-Input
```

**Best Practices:**
- ✅ Immer sinnvolle Defaults angeben
- ✅ Description für komplexe Optionen
- ✅ Validation für numerische Werte
- ✅ Multiple-Choice wenn Optionen begrenzt
- ❌ Nicht für Secrets (nutze `secrets` Feld)
- ❌ Nicht für zu viele Fragen (max. 5)

### 8. Docker-Images

```yaml
# ✅ Offizielle Images bevorzugen
image:
  name: "postgres"              # Docker Hub Official
  name: "mariadb"               # Docker Hub Official
  name: "gitea/gitea"           # Verified Publisher

# ✅ ARM64-Support prüfen (Raspberry Pi!)
# Viele Images haben ARM64-Varianten

# ⚠️  Warnung bei experimentellem ARM64-Support
warnings:
  - "ARM64-Support experimentell - kann instabil sein"

# ❌ Nicht: Unbekannte/unsichere Quellen
image:
  name: "randomuser/sketchy-image"  # VERMEIDEN
```

---

## Checkliste vor Generierung

- [ ] Katalog abgerufen? (`https://apps.kibox.online/containers/`)
- [ ] Port-Konflikte geprüft?
- [ ] Dependencies im Katalog vorhanden?
- [ ] Kategorie korrekt?
- [ ] Alle Pflichtfelder ausgefüllt?
- [ ] Secrets vs. Optional-Secrets korrekt getrennt?
- [ ] Post-Install-Messages hilfreich?
- [ ] Volumes in `/opt/itbox/...`?
- [ ] Docker-Image ARM64-kompatibel?

---

## Antwort-Format an User

Nach Generierung einer Container-YAML:

```markdown
Ich habe eine Container-Definition für [NAME] erstellt:

**Geprüft:**
- ✅ Ports: [PORTS] (frei im Katalog)
- ✅ Dependencies: [DEPS] (vorhanden im Katalog)
- ✅ Kategorie: [CATEGORY]

**Hinweise:**
- [Eventuelle Warnungen]

**Nächste Schritte:**
1. Datei speichern als `[id].yaml`
2. Validieren: `catalogctl validate [id].yaml`
3. Importieren: `sudo catalogctl import [id].yaml`

[YAML-Content hier]
```

---

## Häufige Container-Typen

### Web-Anwendung (Standard)

```yaml
ports:
  - host: 83XX          # HTTP-Port
    container: 80
    protocol: tcp

volumes:
  - host: "/opt/itbox/data/[id]"
    container: "/var/www/html"
```

### Mit Datenbank-Dependency

```yaml
depends_on:
  - mariadb

environment:
  - key: "DB_HOST"
    value: "mariadb:3306"
  - key: "DB_PASSWORD"
    value: "${[ID]_DB_PASSWORD}"

secrets:
  - key: "[ID]_DB_PASSWORD"
    generate: "password"
```

### Game-Server

```yaml
category: games

ports:
  - host: 84XX
    container: 25565
    protocol: udp        # Oft UDP!
    description: "Game-Port"

volumes:
  - host: "/opt/itbox/data/[id]"
    container: "/data"
    description: "Welten und Spieler-Daten"
```

### LMS/Pädagogisch

```yaml
category: education

resources:
  ram_mb: 512          # Oft mehr RAM
  cpu_cores: 1.0
  disk_mb: 200
```

---

## Versionierung

Beim Update eines Containers:

```yaml
# Alte Version
version: "1.0"

# Neue Version (Breaking Changes)
version: "2.0"

# Neue Version (Features)
version: "1.1"

# Bugfixes
version: "1.0.1"
```

User sollte dann re-importieren:
```bash
sudo catalogctl import [container].yaml
```

---

## Troubleshooting-Guide für User

Am Ende jeder YAML immer erwähnen:

```markdown
**Bei Problemen:**
- Validierung: `catalogctl validate [id].yaml`
- Port-Konflikte: Katalog prüfen unter https://apps.kibox.online/containers/
- Dependencies: Erst Dependency importieren
```

---

## Zusammenfassung

1. **Katalog IMMER abrufen** vor Generierung
2. **Ports prüfen** (Konflikte vermeiden)
3. **Dependencies checken** (im Katalog vorhanden?)
4. **Vollständige YAML** generieren (alle Felder)
5. **Best Practices** befolgen (Volumes, Secrets, etc.)
6. **User informieren** über nächste Schritte

**Ziel:** Container-Definitionen, die beim ersten Import funktionieren!
