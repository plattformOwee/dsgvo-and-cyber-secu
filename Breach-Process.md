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