### “Breach-Process” – was die DSGVO in Art. 33 verlangt und wie du es **konkret umsetzt**

| DSGVO-Pflicht                                   | Bedeutung                                                                                                     | Praktische Umsetzung in deinem 2-Person-Startup                                                                                                                                                                                                                         |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Meldung an die Aufsichtsbehörde binnen 72 h** | Sobald „personen­bezogene Daten unbeabsichtigt offengelegt, verändert oder verloren“ wurden.                  | 1) **Uhr startet** in dem Moment, in dem ihr _den Vorfall bemerkt_ (nicht wenn er passiert ist).2) Wenn ihr nach erster Prüfung _wahrscheinlich_ melde­pflichtig seid → Formular der Berliner Behörde (oder deines Bundeslands) abschicken (E-Mail oder Online-Portal). |
| **Benachrichtigung der Betroffenen** (Art. 34)  | Nur wenn hohes Risiko für die Rechte der Nutzer besteht (z. B. Klartext-Passwörter oder Chat-Inhalte publik). | Push + E-Mail-Vorlage vorbereiten, fallback: Banner im Login-Screen.                                                                                                                                                                                                    |

---

## 1 Minimaler **Incident-Run-Book** (One-Pager)

> Lege die Datei `security/incident_runbook.md` in dein Repo – alle wissen sofort, was zu tun ist.

```markdown
# 🔥 Incident Response – Data Breach (v1.0 / 2025-06-11)

## 0. Severity Check
| Stufe | Beispiel | Meldung? |
|-------|----------|----------|
| SEV-0 | Developer gelöscht Staging-DB (keine Prod-Daten) | Nein |
| SEV-1 | Zugriff auf Prod-DB, Hash-Passwörter ausgespäht | Ja, Art. 33 innerhalb 72 h |

## 1. First 30 min
1. PagerDuty/Slack `#critical-incident` alarmiert  
2. Incident Lead bestimmt (wer den Call leitet)  
3. System-Zugriffe einfrieren (read-only)  

## 2. Containment
* Logs sichern (`/var/log/mongodb/*`, CloudTrail)  
* API-Keys rotieren (`AWS`, `SendGrid`)  
* Zugang für kompromittierte IAM-User sperren  

## 3. Impact-Analyse (max 4 h)
* Welche Collections / Buckets betroffen?  
* Welche Datentypen? (E-Mail, Nachrichten, Hash-PW?)  
* Anzahl betroffener User ≈ …  

## 4. Entscheid: Meldepflicht?
* **Ja** → Art. 33 Report (siehe Template) ausfüllen & absenden (< 72 h)  
* **Nein** → internen Report archivieren („close call“)  

## 5. Post-Mortem (D+1)
* Ursachenanalyse, Fix, Retrospektive, Lessons Learned  
* RoPA & TOMs aktualisieren
```

---

## 2 “_critical-incident_” Slack-Channel

- **Privater Channel** (`#incident-<date>`) – Sichtbar nur für Core-Team & externen DSB.
    
- **Slack App Webhook**: `POST /hook/slack-infosec` aus deinem Monitoring, wenn > 100 failed logins/min oder AWS GuardDuty High Alert.
    
- Pinned Items:
    
    - Run-Book PDF
        
    - Link zum 72-h-Formular der Aufsicht
        
    - Vorlage für Benutzer-E-Mail
        
- versucht zu machen aber getenv funcitoniert nicht:
- ## 📂 Datei- & Aufgabenübersicht — „Slack-Incident-Alert“

> **Stand 20 Jun 2025 18:15 UTC**

---

### 🗂️ Neu / geändert – Verzeichnis & Dateien

|Pfad (relativ zu **webroot/**)|Typ|Zweck / Änderung|
|---|---|---|
|`src/Security/slack_alert.php`|**NEU**|Enthält `sendSlackAlert()` (cURL-Post an Slack) + Debug-Log, falls kein Webhook gesetzt.|
|`swipe_chatt_play_api/bootstrap.php`|**GEÄNDERT**|• Composer-Autoload-Pfad gefixt• `dbg()`-Helper• RS256-Validator (`SwipeChatPlay\Security\JwtValidator`)• `bearerToken()`• **TODO:** an erster Stelle `require_once __DIR__.'/../secure/env.php';` einfügen, sobald Datei existiert.|
|`swipe_chatt_play_api/new_signup/verify_code.php`|**GEÄNDERT**|+ Brute-Force-Schutz (pro User & pro IP)+ Lock-Mechanik (5 Fehlversuche/10 min) → HTTP 429+ Slack-Alert bei Lock & bei >30 Fehlversuche/10 min pro IP+ Einbindung `slack_alert.php` (`require_once __DIR__.'/../../src/Security/slack_alert.php';`)|
|`.gitignore`|**GEÄNDERT**|Zeile `*.env` bzw. explizit `secure/env.php` eingetragen → bleibt aus Git heraus.|
|`security/incident_runbook.md`|**NEU**|One-Pager: Incident-Ablauf / 72-h-Checkliste.|
|**(optional)** `monitoring/login_fail_watcher.php`|**NEU**|Cron-Skript-Beispiel: log-tail & Slack-Alert bei mass. Fehlversuchen. _Noch nicht produktiv eingebunden._|

---

### 🛠️ System-/Code-Status

|Bereich|Status|
|---|---|
|**Brute-Force-Protection**|✅ aktiv (User + IP)|
|**Slack-Alert-Funktion**|✅ vorhanden — ruft Webhook auf, **aber** ENV-Variable fehlt ⇒ übersprungen.|
|**`.env` / Dotenv**|🚫 Composer-Install nicht möglich → **Workaround erforderlich**.|
|**JWT-Handling**|✅ RS256, zentral in `bootstrap.php`.|
|**Rate-Limiter-Sammlung**|✅ `ip_fails` (mongod) mit TTL-Index (expireAfterSeconds = 600) – bitte einmal Index setzen: `db.ip_fails.createIndex({ created: 1 }, { expireAfterSeconds: 600 })`.|

---

### 📝 Nächste Schritte (siehe auch To-Do-Block gestern)

1. **secure/env.php** _(ausserhalb Webroot, nicht über HTTP erreichbar)_
    
    ```php
    <?php
    putenv('SLACK_HOOK_CRITICAL=https://hooks.slack.com/services/…');
    putenv('JWT_ISSUER=http://node02.krasserserver.com:8002');
    // weitere Secrets hier …
    ```
    
    - Speicherort-Vorschlag: `/home/container/secure/env.php`
        
    - Rechte: `chmod 600 env.php`
        
2. `bootstrap.php` ganz oben:
    
    ```php
    require_once dirname(__DIR__) . '/secure/env.php';
    ```
    
3. **Test‐Flow**
    
    1. Code 6× falsch eingeben → Response 429
        
    2. Server-Log sollte _nicht_ mehr „ENV not set“ zeigen
        
    3. Slack-Channel `#critical-incident` soll Meldung erhalten.
        
4. Wenn Slack klappt → Monitoring-Cron (`monitoring/login_fail_watcher.php`) auf dem Panel als geplanter Job eintragen.
    

---

### “Copy-Paste-Erinnerung” für den nächsten Arbeitstag 🗒️

```text
[ ] secure/env.php anlegen (Webhook + Issuer)
[ ] require_once in bootstrap.php hinzufügen
[ ] chmod 600 secure/env.php
[ ] Fehlversuche testen → Slack-Alert muss ankommen
[ ] TTL-Index auf ip_fails in Mongo setzen
[ ] Login-Fail-Watcher als Cron/Task planen
```

Damit weißt du sofort, wo wir stehen und was als Nächstes getan wird. ✌️

---

## 3 72-h-Formular – Kurz-Template (copy/paste)

|Abschnitt (Pflichtfelder)|Schnelles Beispiel|
|---|---|
|**A) Verantwortlicher**|MyApp UG, Musterstr. 1, Berlin|
|**B) Beschreibung des Vorfalls**|„23 MB Chat-Dump über öffentliches S3-Bucket zugänglich von 09:15 – 10:02 UTC; Ursache: falsche ACL.“|
|**C) Kategorien betroffener Daten**|Nachrichten-Inhalte, E-Mail-Adressen (ca. 3 500 Nutzer)|
|**D) Folgen & Risiken**|Mögliche Offenlegung privater Chats → hohes Risiko → Art. 34 Benachrichtigung erforderlich|
|**E) Gegenmaßnahmen**|ACL geändert, S3 Inventory Scan, Passwörter zurückgesetzt|
|**F) Kontakt**|Max Mustermann, CTO, +49 30 …|

---

## 4 Tools & Automatisierung

- **AWS GuardDuty + Security Hub** → automatischer Slack-Alert.
    
- **MongoDB Ops Manager** → „Unauthorized access attempts“ → Webhook.
    
- **Backup-Encryption Check** → nightly CI job; fails if unencrypted dump exists.
    

---

## 5 Dokumentation im RoPA / TOMs

Spalte „Technische & organisatorische Maßnahmen“ ergänzt um:

- _“Incident-Run-Book mit 72-h-Prozess, Slack Alerting, jährliches Pen-Test”_
    
- _“Data-at-rest AES-256, transport TLS 1.3, least-privilege IAM”_
    

---

### TL;DR

- **Art. 33** = < 72 h Meldung an Behörde, sobald _meldepflichtiger_ Datenverlust.
    
- Halte eine **Checkliste, Slack-Channel, Formular-Template** bereit – kein Chaos im Ernstfall.
    
- Speichere jeden Zwischen­schritt (Zeitstempel) → beweist, dass ihr die Frist eingehalten habt.
    

Wenn du ein fertiges **Markdown-Run-Book**, eine **Slack Webhook-Snippet** oder ein **AWS Lambda GuardDuty-Alert Beispiel** brauchst – einfach Bescheid geben!