# gtwy - Testing Guide

Vollständiger Test-Plan vor Produktiveinsatz.

---

## 🚀 Quick Start

**Installation in 3 Schritten:**

```bash
# 1. Download gtwy auf Server
scp gtwy root@server:/tmp/
ssh root@server "chmod +x /tmp/gtwy"

# 2. Install (einmalig - richtet alles ein)
ssh root@server "sudo /tmp/gtwy install"

# 3. Setup (konfigurieren)
ssh root@server "sudo gtwy setup"

# ✓ Fertig! Jetzt Boxen registrieren:
ssh root@server "sudo gtwy add-box <box-id> <domain> <public-key>"
```

**Was macht `install`?**
- Kopiert gtwy nach `/opt/gtwy/gtwy`
- Erstellt Benutzer `tunneluser` & Gruppe `gtwy-admin`
- Richtet SSH-Zugriff ein
- Erstellt sudo-Regeln für nginx/certbot
- Setzt alle Permissions korrekt
- Erstellt Symlink → `gtwy` global verfügbar

**Was macht `setup`?**
- Interaktiver Wizard für IONOS API, Domains, Ports, SSL
- Erstellt `/opt/gtwy/config.yml`
- Initialisiert Datenbank `/opt/gtwy/tunnels.db`

---

## 🎯 Test-Strategie

### Phase 1: Lokale Syntax-Tests (auf deinem Mac)
### Phase 2: Server-Setup & Dry-Run
### Phase 3: Test-Domain-Tests (mit echtem IONOS)
### Phase 4: Produktions-Rollout

---

## Phase 1: Lokale Syntax-Tests ✅

### 1.1 Python Syntax Check
```bash
# Auf deinem Mac
cd "/Users/daniel/Developer/IONOS Server"

# Syntax-Check
python3 -m py_compile gtwy

# Import-Test
python3 -c "import yaml, requests; print('Dependencies OK')"
```

**Erwartetes Ergebnis:** Keine Fehler

---

### 1.2 Help-Output Test
```bash
# Hilfe anzeigen (funktioniert ohne Config)
python3 gtwy --help
python3 gtwy setup --help
python3 gtwy add-box --help
```

**Erwartetes Ergebnis:** Help-Text wird angezeigt

---

## Phase 2: Server-Setup & Dry-Run 🚀

### 2.1 gtwy auf Server kopieren
```bash
# Von deinem Mac aus
SERVER="root@your-gateway-server.com"

# gtwy Script hochladen
scp gtwy $SERVER:/tmp/

# Ausführbar machen
ssh $SERVER "chmod +x /tmp/gtwy"
```

---

### 2.2 Installation ausführen
```bash
ssh $SERVER

# gtwy installieren (einmalig!)
# Dies erstellt /opt/gtwy, Benutzer, Gruppen, Permissions, etc.
sudo /tmp/gtwy install

# Was passiert beim Install:
# ✓ Erstellt /opt/gtwy Verzeichnis
# ✓ Kopiert sich selbst nach /opt/gtwy/gtwy
# ✓ Erstellt nginx-template
# ✓ Erstellt Gruppe gtwy-admin
# ✓ Erstellt User tunneluser
# ✓ Richtet SSH-Verzeichnis ein
# ✓ Erstellt sudo-Regeln
# ✓ Erstellt Symlink /usr/local/bin/gtwy
# ✓ Prüft Python-Dependencies
```

---

### 2.2b Dependencies installieren (falls nötig)
```bash
# Falls install Warnung zeigt:
pip3 install PyYAML requests

# ODER auf Debian/Ubuntu:
apt-get update
apt-get install -y python3-yaml python3-requests
```

---

### 2.3 Prerequisite Check
```bash
# Prüfe ob alle Tools verfügbar sind
which python3    # sollte vorhanden sein
which nginx      # sollte vorhanden sein
which certbot    # sollte vorhanden sein
which sqlite3    # sollte vorhanden sein
which gtwy       # sollte jetzt /usr/local/bin/gtwy zeigen

# Nginx-Version
nginx -v

# Certbot-Version
certbot --version

# Python-Version (mind. 3.8)
python3 --version

# gtwy Version
gtwy version
```

---

## Phase 3: Test-Domain-Tests 🧪

### 3.1 IONOS API-Keys generieren

**Im IONOS-Portal:**
1. Login bei IONOS
2. Developer Tools → API Keys
3. "Create New Key" klicken
4. Label: `gtwy-testing`
5. **Public Prefix** und **Secret** kopieren

**⚠️ WICHTIG:** Für Tests **STAGING-MODE** nutzen!

---

### 3.2 Setup-Wizard ausführen
```bash
# Interaktive Konfiguration
sudo gtwy setup

# Eingaben:
# IONOS API Configuration:
#   Public Prefix: abc123_
#   Secret:        xyz789secret
#
# Server Configuration:
#   Public IP:     [auto-detected oder manuell]
#   Hostname:      gateway.kibox.online (optional)
#
# Domain Configuration:
#   Allowed domains: test.kibox.online (oder leer für alle)
#
# Port Ranges:
#   Service port start [10000]: 10000
#   Service port end [19999]:   10099  (klein für Tests)
#   Admin SSH port start [20000]: 20000
#   Admin SSH port end [20999]:   20099  (klein für Tests)
#
# SSL Configuration:
#   Certbot email: deine-email@example.com
#   Use staging:   y  (WICHTIG für Tests!)
```

**Erwartetes Ergebnis:**
```
✓ Configuration written to /opt/gtwy/config.yml
✓ Database initialized at /opt/gtwy/tunnels.db
✓ Directories created
✓ Permissions set
✓ Setup complete!

Next steps:
  gtwy add-box <box-id> <domain> <public-key>
```

---

### 3.3 Health-Check
```bash
gtwy health
```

**Erwartetes Ergebnis:**
```
System Health Check:

Database:       ✓ OK (0 boxes, 0 tunnels)
Config:         ✓ OK
nginx:          ✓ running
certbot:        ✓ installed
IONOS API:      ✓ reachable
Disk Space:     ✓ XX% free
Permissions:    ✓ OK

Overall: ✓ HEALTHY
```

**Falls IONOS API FAIL:** API-Keys prüfen!

---

### 3.4 Test-Box vorbereiten

**Auf einer Test-Box (kann auch dein Mac sein):**
```bash
# SSH-Key generieren
ssh-keygen -t ed25519 -f ~/.ssh/test-tunnel-key -C "test-box"

# Public Key anzeigen
cat ~/.ssh/test-tunnel-key.pub
# Kopiere den Output
```

---

### 3.5 Test-Box registrieren
```bash
# Auf dem Gateway-Server
sudo gtwy add-box test-box01 test.kibox.online "ssh-ed25519 AAAAC3..." --notes "Test Box"

# Output prüfen:
# ✓ Box test-box01 added successfully
#   Domain:       test.kibox.online
#   Admin SSH:    Port 20001
#   ...
```

**Verifikation:**
```bash
# Box sollte in Liste sein
gtwy list-boxes

# authorized_keys prüfen
cat /home/tunneluser/.ssh/authorized_keys
# Sollte Eintrag für test-box01 enthalten

# Port abrufen
gtwy get-port test-box01
# Sollte: 20001
```

---

### 3.6 SSH-Verbindung testen

**Von der Test-Box:**
```bash
# Verbindung testen
ssh -i ~/.ssh/test-tunnel-key tunneluser@gateway.kibox.online

# Sollte Fehler geben (command restriction):
# bash: BOX_ID=test-box01 /opt/gtwy/gtwy request: command not found

# Das ist KORREKT! Bedeutet SSH funktioniert.
```

---

### 3.7 Tunnel-Request testen

**WICHTIG:** Dieser Test erstellt echte DNS-Records (in Staging)!

**Von der Test-Box:**
```bash
# Tunnel anfordern
PORT=$(ssh -i ~/.ssh/test-tunnel-key tunneluser@gateway.kibox.online "request test-service 8080")
echo "Allocated port: $PORT"

# Sollte Port zurückgeben, z.B.: 10001
```

**Auf dem Gateway prüfen:**
```bash
# Tunnel sollte in DB sein
gtwy list

# Sollte zeigen:
# test-service.test-box01.test.kibox.online | test-box01 | test-service | 8080 | 10001 | pending_cert | ...

# nginx-Config prüfen
cat /etc/nginx/sites-enabled/tunnels-autogen.conf
# Sollte Eintrag für test-service.test-box01.test.kibox.online enthalten

# DNS-Record prüfen (nach 1-2 Min)
dig test-service.test-box01.test.kibox.online
# Sollte A-Record zeigen
```

---

### 3.8 Zertifikat-Request testen

**⚠️ WICHTIG:** Wir nutzen Staging-Mode!

```bash
# Warte 2-3 Minuten für DNS-Propagation
sleep 180

# Zertifikat anfordern
sudo gtwy sync-certs

# Output prüfen:
# test-service.test-box01.test.kibox.online:
#   Checking DNS propagation... ✓
#   Requesting certificate... ✓
#   Status updated: active
```

**Falls DNS-Propagation fehlschlägt:** Länger warten (bis zu 5 Min)

---

### 3.9 Tunnel info & test

```bash
# Detaillierte Info
gtwy info test-service.test-box01.test.kibox.online

# End-to-End-Test
gtwy test-tunnel test-service.test-box01.test.kibox.online

# Sollte zeigen:
# ✓ DNS resolution
# ✓ Port listening
# ✓ Certificate exists
```

---

### 3.10 Tunnel-Release testen

**Von der Test-Box:**
```bash
ssh -i ~/.ssh/test-tunnel-key tunneluser@gateway.kibox.online "release test-service"

# Sollte: ✓ Tunnel test-service released
```

**Auf dem Gateway prüfen:**
```bash
# Tunnel sollte weg sein
gtwy list
# Sollte leer sein oder Tunnel nicht mehr zeigen

# nginx-Config prüfen
cat /etc/nginx/sites-enabled/tunnels-autogen.conf
# Sollte LEER sein oder Eintrag entfernt
```

---

### 3.11 Box entfernen testen

```bash
# Box entfernen
sudo gtwy remove-box test-box01 -y

# Prüfen
gtwy list-boxes
# Sollte leer sein

# authorized_keys prüfen
cat /home/tunneluser/.ssh/authorized_keys
# Sollte Eintrag für test-box01 NICHT mehr enthalten
```

---

### 3.12 Backup/Restore testen

```bash
# Backup erstellen
sudo gtwy backup

# Backup-Datei prüfen
ls -lh /opt/gtwy/backups/
# Sollte gtwy-backup-YYYY-MM-DD-HHMMSS.tar.gz zeigen

# Backup-Inhalt prüfen
tar -tzf /opt/gtwy/backups/gtwy-backup-*.tar.gz
# Sollte: tunnels.db, config.yml, nginx-template, authorized_keys

# Restore testen (ACHTUNG: Überschreibt DB!)
sudo gtwy restore /opt/gtwy/backups/gtwy-backup-*.tar.gz --force
```

---

## Phase 4: Produktions-Rollout 🚀

### 4.1 Staging → Production umstellen

**Config anpassen:**
```bash
sudo nano /opt/gtwy/config.yml

# Ändern:
certbot:
  email: admin@kibox.online
  staging: false  # ← von true auf false!

domains:
  - kibox.online          # ← echte Domains
  - itbox.niedersachsen.de

port_range:
  services:
    start: 10000
    end: 19999    # ← volle Range
  admin_ssh:
    start: 20000
    end: 20999    # ← volle Range
```

---

### 4.2 Staging-Zertifikate aufräumen

```bash
# Staging-Certs löschen (falls welche erstellt wurden)
sudo certbot certificates
# Zeigt alle Zertifikate

# Staging-Certs löschen
sudo certbot delete --cert-name test-service.test-box01.test.kibox.online
```

---

### 4.3 Erste Produktions-Box

```bash
# Echte Box registrieren
sudo gtwy add-box box01 kibox.online /path/to/real-box-key.pub

# Tunnel von Box anfordern (siehe 3.7)

# Zertifikat-Cronjob einrichten
crontab -e

# Hinzufügen:
*/5 * * * * /opt/gtwy/gtwy sync-certs >> /opt/gtwy/cron.log 2>&1
```

---

### 4.4 Monitoring einrichten

```bash
# Status-Check-Cronjob (optional)
crontab -e

# Hinzufügen:
*/15 * * * * /opt/gtwy/gtwy status >> /opt/gtwy/status.log 2>&1

# Backup-Cronjob (täglich)
0 2 * * * /opt/gtwy/gtwy backup >> /opt/gtwy/backup.log 2>&1
```

---

## 🔍 Troubleshooting

### Problem: "IONOS API error: UNAUTHORIZED"
**Lösung:**
```bash
# API-Keys prüfen
cat /opt/gtwy/config.yml | grep -A3 ionos

# Test-Request
curl -H "X-API-Key: PREFIX.SECRET" https://api.hosting.ionos.com/dns/v1/zones
```

---

### Problem: "nginx configuration test failed"
**Lösung:**
```bash
# nginx manuell testen
sudo nginx -t

# Config prüfen
cat /etc/nginx/sites-enabled/tunnels-autogen.conf

# Syntax-Fehler im Template?
cat /opt/gtwy/nginx-template
```

---

### Problem: "Certificate request failed"
**Lösung:**
```bash
# DNS prüfen
dig subdomain.box.domain.com

# Manueller certbot-Test
sudo certbot certonly --nginx -d subdomain.box.domain.com --dry-run

# Log prüfen
tail -f /var/log/letsencrypt/letsencrypt.log
```

---

### Problem: "Database locked"
**Lösung:**
```bash
# Prüfe ob andere gtwy-Prozesse laufen
ps aux | grep gtwy

# Falls ja, killen oder warten

# DB-Permissions prüfen
ls -l /opt/gtwy/tunnels.db
# Sollte: tunneluser:gtwy-admin
```

---

## ✅ Test-Checkliste

### Phase 1: Lokal ✅
- [ ] Python Syntax Check
- [ ] Dependencies installiert
- [ ] Help-Output funktioniert

### Phase 2: Server-Setup ✅
- [ ] Dateien auf Server
- [ ] Dependencies installiert
- [ ] Benutzer & Permissions
- [ ] Sudo-Regeln
- [ ] Setup-Wizard erfolgreich
- [ ] Health-Check grün

### Phase 3: Test-Domain ✅
- [ ] IONOS API erreichbar
- [ ] Test-Box registriert
- [ ] SSH-Verbindung funktioniert
- [ ] Tunnel-Request funktioniert
- [ ] nginx-Config generiert
- [ ] DNS-Record erstellt
- [ ] Zertifikat ausgestellt (Staging)
- [ ] Tunnel-Release funktioniert
- [ ] Box-Removal funktioniert
- [ ] Backup/Restore funktioniert

### Phase 4: Produktion ✅
- [ ] Staging → Production umgestellt
- [ ] Erste echte Box registriert
- [ ] Echtes Zertifikat ausgestellt
- [ ] Cronjobs eingerichtet
- [ ] Monitoring läuft

---

## 🎓 Best Practices

1. **Immer mit Staging testen** bevor echte Zertifikate angefordert werden
2. **Backups vor größeren Änderungen** erstellen
3. **Logs regelmäßig prüfen:** `/opt/gtwy/gtwy.log`
4. **Health-Checks einrichten** für Monitoring
5. **DNS-Propagation** kann 5-10 Minuten dauern
6. **Let's Encrypt Rate Limits beachten** (5 Certs/Woche pro Domain)

---

## 📚 Weiterführende Docs

- **IONOS API:** https://developer.hosting.ionos.de/docs/dns
- **Let's Encrypt Staging:** https://letsencrypt.org/docs/staging-environment/
- **nginx Proxy:** https://nginx.org/en/docs/http/ngx_http_proxy_module.html
- **certbot Docs:** https://eff-certbot.readthedocs.io/

---

**Viel Erfolg beim Testing! 🚀**
