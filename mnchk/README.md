# mnchk - moin.schule Credential Checker

Kompaktes CLI-Tool zur Validierung von moin.schule Credentials.

## Features

- ✅ **Einfache Installation**: `sudo mnchk install`
- ✅ **Systemweit verfügbar**: Nach Installation für alle Benutzer nutzbar
- ✅ **Maschinenlesbar**: Exit-Codes + optionaler JSON-Output
- ✅ **Script-freundlich**: Ideal für Automatisierung und Integration
- ✅ **Minimale Dependencies**: Nur `python3-requests` erforderlich

## Installation

```bash
# 1. Dependency installieren (falls nicht vorhanden)
sudo apt install python3-requests

# 2. mnchk installieren
sudo ./mnchk install
```

Nach der Installation ist `mnchk` systemweit unter `/usr/local/bin/mnchk` verfügbar.

## Verwendung

### Passwort-Eingabemethoden

**1. Interaktiver Modus (EMPFOHLEN):**
```bash
mnchk --user max.mustermann
# Password for max.mustermann: [Eingabe versteckt]
```

**2. Via stdin (SICHER - kein History-Problem):**
```bash
echo "geheim123" | mnchk --user max.mustermann --password-stdin
# oder mit Here-String:
mnchk --user max.mustermann --password-stdin <<< "geheim123"
```

**3. Via Datei (SICHER):**
```bash
echo "geheim123" > /tmp/pass
mnchk --user max.mustermann --password-file /tmp/pass
rm /tmp/pass
```

**4. Via Argument (NUR FÜR TESTS - UNSICHER!):**
```bash
mnchk --user max.mustermann --password geheim123
# ⚠️ WARNUNG: Passwort erscheint in bash history!
# ⚠️ PROBLEM: Passwörter mit ! führen zu Bash-Fehlern!
```

**Output:**
- `true` → Credentials gültig (Exit Code: 0)
- `false` → Credentials ungültig (Exit Code: 1)

### JSON Output

```bash
mnchk --user max.mustermann --password-stdin --json <<< "geheim123"
```

**Output bei gültigem Account:**
```json
{
  "valid": true,
  "status": "success",
  "message": "Credentials valid, account complete",
  "user": "max.mustermann"
}
```

**Output bei unvollständigem Account (aber korrektem Passwort):**
```json
{
  "valid": true,
  "status": "incomplete_account",
  "message": "Credentials valid, account setup incomplete",
  "user": "max.mustermann"
}
```

**Output bei falschen Credentials:**
```json
{
  "valid": false,
  "status": "invalid_credentials",
  "message": "Invalid username or password",
  "user": "max.mustermann"
}
```

## Integration in Scripts

### Bash Script

**SICHERE Methode (via stdin):**
```bash
#!/bin/bash

USER="max.mustermann"
PASS="geheim123"

# Methode 1: Here-String (EMPFOHLEN)
if mnchk --user "$USER" --password-stdin <<< "$PASS"; then
    echo "✓ Login erfolgreich"
else
    echo "✗ Login fehlgeschlagen"
    exit 1
fi

# Methode 2: Pipe
if echo "$PASS" | mnchk --user "$USER" --password-stdin; then
    echo "✓ Login erfolgreich"
fi

# Methode 3: Interaktiv (für User-Eingabe)
if mnchk --user "$USER"; then
    echo "✓ Login erfolgreich"
fi
```

**UNSICHERE Methode (nur für Tests):**
```bash
# ⚠️ NICHT VERWENDEN: Passwort in Command Line sichtbar!
if mnchk --user "$USER" --password "$PASS"; then
    echo "Login OK"
fi
```

### Python Script

**SICHERE Methode (via stdin):**
```python
import subprocess
import json

def check_moin_credentials(username, password):
    """Check moin.schule credentials (SECURE)"""
    result = subprocess.run(
        ['mnchk', '--user', username, '--password-stdin', '--json'],
        input=password,
        capture_output=True,
        text=True
    )

    if result.returncode == 0:
        data = json.loads(result.stdout)
        return data['valid']
    return False

# Verwendung
if check_moin_credentials('max.mustermann', 'geheim123'):
    print("Login OK")
else:
    print("Login failed")
```

**Alternative: Interaktiv**
```python
import subprocess

result = subprocess.run(
    ['mnchk', '--user', 'max.mustermann'],
    # stdin bleibt am Terminal, User kann Passwort eingeben
)

if result.returncode == 0:
    print("Login OK")
```

### PHP Script

**SICHERE Methode (via stdin):**
```php
<?php
function checkMoinCredentials($username, $password) {
    $descriptors = [
        0 => ["pipe", "r"],  // stdin
        1 => ["pipe", "w"],  // stdout
        2 => ["pipe", "w"]   // stderr
    ];

    $cmd = sprintf(
        'mnchk --user %s --password-stdin --json',
        escapeshellarg($username)
    );

    $process = proc_open($cmd, $descriptors, $pipes);

    if (is_resource($process)) {
        // Password via stdin schreiben
        fwrite($pipes[0], $password);
        fclose($pipes[0]);

        $output = stream_get_contents($pipes[1]);
        fclose($pipes[1]);
        fclose($pipes[2]);

        $exitCode = proc_close($process);

        if ($exitCode === 0) {
            $result = json_decode($output, true);
            return $result['valid'];
        }
    }

    return false;
}

// Verwendung
if (checkMoinCredentials('max.mustermann', 'geheim123')) {
    echo "Login OK\n";
} else {
    echo "Login failed\n";
}
?>
```

## Exit Codes

| Code | Bedeutung |
|------|-----------|
| 0 | Credentials gültig |
| 1 | Credentials ungültig |
| 2 | Fehler (Netzwerk, fehlende Parameter, etc.) |

## Funktionsweise

Das Tool nutzt den ROPC (Resource Owner Password Credentials) Flow von Keycloak mit einem Trick:

**Keycloak prüft Credentials VOR dem Account-Status:**

1. ✅ **Status 200**: Passwort korrekt + Account vollständig eingerichtet
2. ✅ **"account is not fully set up"**: Passwort korrekt + Account unvollständig
3. ❌ **"invalid user credentials"**: Passwort falsch

**Für Login-Zwecke gelten Fall 1 + 2 als gültig!**

Dies entspricht dem Verhalten des originalen [ldap_password_validator.py](moin.schule Experimente/ldap_password_validator.py).

## Konfiguration

Die Produktiv-Endpoints sind fest im Code konfiguriert:

```python
TOKEN_ENDPOINT = "https://auth.moin.schule/realms/moins/protocol/openid-connect/token"
CLIENT_ID = "admin-cli"
```

Für andere Umgebungen (z.B. Staging) muss das Script entsprechend angepasst werden.

## Sicherheitshinweise & Bash History Problem

### Das Ausrufezeichen-Problem

⚠️ **PROBLEM:** Passwörter mit `!` führen zu Bash-Fehlern!

```bash
# Dies funktioniert NICHT:
mnchk --user test --password 'Test!123'
# Fehler: -bash: !123: event not found

# Grund: Bash History Expansion interpretiert ! als Kommando
```

### Lösungen (in Reihenfolge der Empfehlung)

**1. ✅ BESTE LÖSUNG: Interaktiver Modus**
```bash
mnchk --user max.mustermann
# Passwort wird versteckt abgefragt (kein Echo)
# Funktioniert mit ALLEN Sonderzeichen
# Erscheint NICHT in bash history
```

**2. ✅ GUT: stdin-Methode**
```bash
# Aus Umgebungsvariable:
export PASS="Test!123"
mnchk --user max --password-stdin <<< "$PASS"
unset PASS

# Oder via Pipe:
echo "Test!123" | mnchk --user max --password-stdin
```

**3. ✅ ALTERNATIV: Datei-Methode**
```bash
# Temporäre Datei (automatisch aufräumen):
PASSFILE=$(mktemp)
echo "Test!123" > "$PASSFILE"
mnchk --user max --password-file "$PASSFILE"
rm -f "$PASSFILE"
```

**4. ❌ NICHT EMPFOHLEN: --password Argument**
```bash
# NUR für einfache Test-Passwörter ohne Sonderzeichen!
mnchk --user test --password abc123

# PROBLEME:
# - Erscheint in bash history
# - Sichtbar in Prozessliste (ps aux)
# - Sonderzeichen (!, $, etc.) verursachen Fehler
```

### Weitere Sicherheitshinweise

- ✅ Passwörter aus Umgebungsvariablen lesen (`$PASS`)
- ✅ Nach Verwendung aufräumen (`unset PASS`)
- ✅ Temporäre Dateien löschen (`rm /tmp/pass`)
- ✅ In Produktiv: HTTPS verwenden
- ❌ Passwörter NIEMALS in Code committen
- ❌ Passwörter NIEMALS in Logfiles schreiben

## Technische Details

- **Sprache**: Python 3
- **Dependencies**: `requests` (python3-requests)
- **Timeout**: 10 Sekunden pro Anfrage
- **Installation**: Kopie nach `/usr/local/bin/`
- **Permissions**: 0755 (ausführbar für alle)

## Deinstallation

```bash
sudo rm /usr/local/bin/mnchk
```

## Changelog

### v1.0.1 (2025-12-15)
- ✨ Neue Features:
  - `--password-stdin` für sichere Passwort-Eingabe
  - `--password-file` zum Lesen aus Datei
  - Interaktiver Modus (getpass) bei fehlendem Passwort
- 🐛 Fix: Bash History Expansion Problem mit Sonderzeichen
- 📚 Erweiterte Dokumentation mit Sicherheitshinweisen

### v1.0.0 (2025-12-15)
- Initial Release
- Basic credential validation
- JSON output support
- Installation via `mnchk install`
- Production endpoint configuration
