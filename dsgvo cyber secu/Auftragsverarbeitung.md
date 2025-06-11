### Auftragsverarbeitung nach Art. 28 DSGVO – Schritt-für-Schritt in deinem Projekt

**Kurz:** Jede externe Firma, die für dich personenbezogene Daten **verarbeitet aber nicht selbst nutzt** (= „Processor“), braucht mit dir einen **Auftragsverarbeitungs-Vertrag (DPA / AVV)**.  
Darin steht u. a. _was_ sie mit den Daten tun dürfen, _welche_ Sicherheitsmaßnahmen gelten und _wer_ haftet. Bei Anbietern außerhalb des EWR musst du zusätzlich die EU-**Standard-Vertragsklauseln (SCC)** einschließen.

---

## 1 · Typische Processor-Liste für deine Stack

|Dienst|Beispielanbieter|Welche Daten fließen?|Sitz / Server-Region|Was du konkret tust|
|---|---|---|---|---|
|**Hosting / Datenbank**|Hetzner Cloud, AWS-eu-central-1|alle Account- & Chatdaten|EU/EWR|• Online-AVV akzeptieren (Hetzner „AG Auftragsverarbeitung“, AWS DPA auto-inkorporiert) ([docs.aws.amazon.com](https://docs.aws.amazon.com/whitepapers/latest/navigating-gdpr-compliance/aws-data-processing-addendum-dpa.html?utm_source=chatgpt.com "AWS Data Processing Addendum (DPA) - AWS Documentation"))|
|**E-Mail-Versand**|SendGrid, Mailgun|Login-Bestätigungen, Newsletter|USA|• Twilio/SendGrid DPA inkl. **SCC** gilt automatisch durch T&C ([sendgrid.com](https://sendgrid.com/en-us/resource/general-data-protection-regulation-2?utm_source=chatgpt.com "General Data Protection Regulation - SendGrid"))|
|**Crash-Reports**|Firebase Crashlytics|anonyme Install-ID, Stacktrace|EU + USA|• In Firebase-Console „Daten­verarbeitung“ bestätigen (DPA + SCC) ([cloud.google.com](https://cloud.google.com/terms/data-processing-addendum?utm_source=chatgpt.com "Cloud Data Processing Addendum \| Google Cloud"))|
|**Push-Benachrichtigungen**|Firebase Cloud Messaging|Geräte-Token|EU + USA|dito|
|**Back-ups / Storage**|AWS S3 (EU), Backblaze B2|verschlüsselte Dumps|EU / USA|• Falls USA → SCC einbeziehen|
|**Monitoring / Logs**|Sentry (EU), Datadog|Fehlermeldungen|EU / USA|• EU-Region bevorzugen, sonst SCC|

> **Controller ≠ Processor:** Stripe (Payment) agiert meist _eigenverantwortlich_ → Koordination per **Joint Controller Addendum**, nicht AVV.

---

## 2 · So schließt du eine DPA in der Praxis

|Schritt|Was genau passiert?|
|---|---|
|**a. Anbieter-Konsole**|Bei Cloud-Riesen klickst du nur eine Checkbox:• **AWS**: Console → Account → “GDPR DPA” (greift automatisch) ([docs.aws.amazon.com](https://docs.aws.amazon.com/whitepapers/latest/navigating-gdpr-compliance/aws-data-processing-addendum-dpa.html?utm_source=chatgpt.com "AWS Data Processing Addendum (DPA) - AWS Documentation"))• **Google Cloud/Firebase**: Console → Account Settings → „Data Processing & Security Terms“ ✔ ([cloud.google.com](https://cloud.google.com/terms/data-processing-addendum?utm_source=chatgpt.com "Cloud Data Processing Addendum \| Google Cloud"))|
|**b. PDF herunterladen**|Lade die signierte Fassung (oder den Nachweis-Screenshot) als **evidence** in deinen Compliance-Ordner.|
|**c. Prüfe TOM-Anhang**|Jeder DPA hat einen Anhang „Technische & Organisatorische Maßnahmen“ (TOM). Notiere relevante Punkte in der RoPA-Spalte „Empfänger / Sicherheitsmaßnahmen“.|
|**d. Nicht-EU?**|Liegt der Haupt-Server in den USA, muss der Anbieter neben dem DPA automatisch die **SCC Module 2 (Controller → Processor)** anhängen. Viele tun das bereits (SendGrid) ([sendgrid.com](https://sendgrid.com/en-us/blog/how-the-recent-privacy-shield-ruling-impacts-twilio-sendgrid-users?utm_source=chatgpt.com "How the Recent Privacy Shield Ruling Impacts Twilio SendGrid Users"))|
|**e. Transfer-Impact-Assessment**|Kurz-Risiko-Check (TIA) beilegen: _„SendGrid speichert nur E-Mail- Adresse & Metadaten; Daten werden TLS-verschlüsselt übertragen; Zugriff via SOC-2 geprüfte Rechenzentren“._|

---

## 3 · Dokumentation im Verarbeitungsverzeichnis (RoPA / VVT)

Ergänze pro Eintrag eine neue Spalte:

|…|**Empfänger**|**Standort**|**Rechtsgrundlage**|**Sicherungsmaßnahmen**|
|---|---|---|---|---|
|Nachrichten-Hosting|Hetzner Online GmbH|Falkenstein (DE)|Art. 28 i. V. m. AVV|ISO 27001, TLS 1.3, Daten auf dedizierten Servern verschlüsselt|

---

## 4 · Pflichten nach Vertragsschluss

1. **„Only on instructions“** – Jeder Processor darf Daten nur so nutzen, wie du es vorgibst.
    
2. **Sub-Processor-Liste** prüfen: AWS → Amazon Inc., SendGrid → Twilio Inc.
    
3. **Audit-Recht** – in der Praxis reicht der jährliche ISO/SOC-Report.
    
4. **Meldung von Data Breaches** – vertraglich meist „unverzüglich, max 24 h“.
    

---

## 5 · Wann brauchst du _kein_ AVV?

|Szenario|Beurteilung|
|---|---|
|**reiner Domain-Registrar** (nur Rechnungsdaten)|Verarbeitung im „eigenen Interesse“ → **eigener Controller**|
|**SMTP-Relay mit inhaltlichem Spam-Filter**|Greift aktiv in Daten ein → **Processor**, also **AVV nötig**|
|**Analytics SDK, das User-Tracking selbst zu Werbezwecken nutzt**|Wird _eigener_ Controller → Kein AVV möglich, stattdessen **Joint Controller** oder _keine Integration_|

---

### To-Do-Liste für dich

1. **Inventar von allen externen Diensten** aufstellen.
    
2. **Download / unterschreiben** aller relevanten DPAs (PDF in `/compliance/DPAs/`).
    
3. **RoPA aktualisieren** – Empfänger, SCC, TOM.
    
4. **Privacy-Policy Abschnitt „Datenempfänger“** um die Provider ergänzen.
    
5. **Vertragspflege-Reminder** (jährlich): Prüfe Versions-Updates der DPAs.
    

Erledigst du diese vier Punkte, deckst du Art. 28 DSGVO sauber ab. Wenn du Hilfe bei einer **TIA-Vorlage** oder einem **Muster-DPA** für kleine europäische Dienstleister brauchst, sag Bescheid!