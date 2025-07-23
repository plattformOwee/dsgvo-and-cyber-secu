**Kurzüberblick**  
Die nachfolgende, erweiterte To‑do‑Liste übersetzt jeden Stichpunkt in ganz konkrete organisatorische oder technische Maßnahmen, die sich direkt aus den einschlägigen Artikeln der DSGVO, den Leitlinien der Aufsichtsbehörden und – wo relevant – Dienstanbieter‑Verträgen ableiten. Damit sehen Sie sofort, **welches Dokument wo abgelegt werden muss, welchen Workflow Sie einrichten sollten und welche Entwicklungs‑Tasks in Ihrem Code‐Base landen**.

---

## 1  Dateninventur & RoPA (Art. 30 DSGVO)

| Aufgabe                            | Konkret zu erledigen                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **RoPA‐Struktur anlegen**          | Erstellen Sie ein elektronisches Register (z. B. Excel‑Sheet oder Confluence‑Seite), das _sämtliche_ Felder aus Art. 30 Abs. 1 enthält: Verantwortlicher + DPO, Zwecke, Kategorien betroffener Personen/Daten, Empfänger, Drittlandtransfers, Löschfristen, TOMs (TOM‑Verweis auf Art. 32) ([GDPR](https://gdpr-info.eu/art-30-gdpr/?utm_source=chatgpt.com "Art. 30 GDPR – Records of processing activities - General Data ..."), [Homepage \| Data Protection Commission](https://www.dataprotection.ie/sites/default/files/uploads/2023-04/Records%20of%20Processing%20Activities%20%28RoPA%29%20under%20Article%2030%20GDPR.pdf?utm_source=chatgpt.com "[PDF] Records of Processing Activities (RoPA) under Article 30 GDPR")) |
| **Inventur‑Workflow**              | • Terminieren Sie einen quartalsweisen Review‑Prozess (Kalendereintrag für DPO/CTO). • Neue Features dürfen erst gemergt werden, wenn ein PR‑Template den Haken „RoPA‑Eintrag ergänzt“ enthält.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| **Abgleich Einwilligung ↔ AGB/PP** | Prüfen Sie pro Verarbeitung, ob Rechtsgrundlage „Einwilligung“ wirklich notwendig ist. Wenn ja, **AGB + Privacy Notice anpassen** (Checkbox‐Flows + Consent‑Log in DB).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| **Vollständigkeits‑Check**         | Implementieren Sie ein Script, das die “collector‑Events” im Code (ProcessingTags) gegen die RoPA‐Zeilen matched und fehlende Zeilen in ein Logfile schreibt.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |

---

## 2  Rechtsgrundlage festlegen

| Aufgabe                           | Konkret zu erledigen                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| --------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Mapping‑Tabelle**               | Legen Sie eine JSON‑ oder YAML‑Datei an, die jede Verarbeitung der passenden Rechtsgrundlage zuordnet: • Vertragsdurchführung → Art. 6 Abs. 1 lit. b (Abo‑Nutzung) ([GDPR](https://gdpr-info.eu/art-6-gdpr/?utm_source=chatgpt.com "Art. 6 GDPR – Lawfulness of processing - General Data Protection ...")) • Optionale Analytics/Mailings → Einwilligung lit. a; Consent‑Banner mit granularer Opt‑In‑Liste.                                                                                                                                                                                                                                         |
| **Legitimate‑Interest‑Bewertung** | Für Crashlytics o. Ä. können Sie „Berechtigtes Interesse“ nutzen → schriftliches _Legitimate Interest Assessment_ (LIA) ablegen; enthält Zweck, Notwendigkeit, Abwägung, Safeguards ([ICO](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/lawful-basis/legitimate-interests/how-do-we-apply-legitimate-interests-in-practice/?utm_source=chatgpt.com "How do we apply legitimate interests in practice? \| ICO"), [EDPB](https://www.edpb.europa.eu/system/files/2024-10/edpb_guidelines_202401_legitimateinterest_en.pdf?utm_source=chatgpt.com "[PDF] Guidelines 1/2024 on processing of personal data based on Article ...")) |
| **DPIA‑Ampel**                    | Ergänzen Sie die Mapping‑Tabelle um eine Spalte „High‑Risk?“. Wenn _true_, löst das Pipeline‑Hook `run_dpia.sh` aus, das ein DPIA‑Markdown‑Template öffnet (Pflicht nach Art. 35) ([GDPR](https://gdpr-info.eu/art-35-gdpr/?utm_source=chatgpt.com "Art. 35 GDPR – Data protection impact assessment"), [European Commission](https://commission.europa.eu/law/law-topic/data-protection/rules-business-and-organisations/obligations/when-data-protection-impact-assessment-dpia-required_en?utm_source=chatgpt.com "When is a Data Protection Impact Assessment (DPIA) required?"))                                                                 |

---

## 3  Privacy Policy & Impressum

| Aufgabe                      | Konkret zu erledigen                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Pflichtangaben**           | Privacy‑Notice muss Angaben aus Art. 13/14 enthalten (Kontaktdaten, Zwecke, Rechtsgrundlagen, Empfänger, Drittlandtransfer, Speicherdauer, Betroffenenrechte, Beschwerderecht) ([GDPR](https://gdpr-info.eu/art-13-gdpr/?utm_source=chatgpt.com "Art. 13 GDPR – Information to be provided where personal data are ..."), [GDPR](https://gdpr-info.eu/art-14-gdpr/?utm_source=chatgpt.com "Art. 14 GDPR – Information to be provided where personal data ..."))                                                                                                  |
| **Ausübung der Rechte**      | Fügen Sie einen Abschnitt „So üben Sie Ihre Rechte aus“ hinzu → E‑Mail privacy@… oder rights‑Formular. Formular speichert Request in `user_rights`‑Collection (Status=PENDING). ([GDPR.eu](https://gdpr.eu/privacy-notice/?utm_source=chatgpt.com "Writing a GDPR-compliant privacy notice (template included)"), [ICO](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/individual-rights/the-right-to-be-informed/what-privacy-information-should-we-provide/?utm_source=chatgpt.com "What privacy information should we provide? \| ICO")) |
| **Versionierung & Nachweis** | Jede Änderung der Notice als Markdown‑File versionieren; Release‑Tag + Commit‑Hash in Footer anzeigen.                                                                                                                                                                                                                                                                                                                                                                                                                                                           |

---

## 4  Crashlytics (Beispiel: Firebase Crashlytics)

|Aufgabe|Konkret zu erledigen|
|---|---|
|**Auftragsverarbeitung**|Google‑DPA unterschreiben und in `contracts/` ablegen ([Firebase](https://firebase.google.com/terms/data-processing-terms?utm_source=chatgpt.com "Firebase Data Processing and Security Terms - Google"), [Firebase](https://firebase.google.com/support/privacy?utm_source=chatgpt.com "Privacy and Security in Firebase - Google"))|
|**SDK‑Konfiguration**|• Deaktivieren Sie User‑Identifiers (setUserId null), bis Consent erteilt. • Setzen Sie `com.google.firebase.crashlytics.analyticsCollectionEnabled` per Remote‑Config.|
|**Opt‑Out Mechanismus**|Toggle im „Permissions‑Page“ → setzt Flag `crashlytics_optout=true`; App ruft `Crashlytics.setCrashlyticsCollectionEnabled(false)`.|
|**RoPA‑Eintrag**|Neue Zeile in RoPA: Verarbeitung „Crash‑Reports“, legitimes Interesse/EU‑DPA, Speicherdauer 90 Tage.|

---

## 5  User‑Rights‑API

|Recht|Backend‑Endpoint & Pflichten|
|---|---|
|**Löschung (delete_me.php)**|• Authentifizierter POST legt Dokument `{userId, createdAt, status}` in `delete_requests`. • CRON‑Job `/scripts/cleanup_delete_requests.php` prüft `status=PENDING && createdAt<now‑30d` und ruft `users.deleteOne()`; 2‑Man‑Approval via `requires_admin_signoff` Boolean.|
|**Datenexport (request_me.php)**|• Generiert ZIP in `/exports/{uuid}.zip` + Presigned‑URL (24 h). • Nach 24 h Cron `cleanup_exports.sh` löscht Datei.|
|**Berichtigung (rectification)**|• Authentifizierte Seite `/account/edit`. Änderungen schreiben sofort in DB; API validiert Felder.|
|**Widerspruch (objection)**|• Permissions‑Page mit Schaltern (Mails, Analytics …). POST `/api/permissions/update`; Änderungen sofort wirksam.|
|**Einschränkung (restriction)**|• Toggle `processing_restricted=true`; Middleware prüft Flag und blockt alle Nicht‑Core‑Jobs. • Entsperren nur per erneutem Toggle. Bedingungen siehe Art. 18 ([GDPR](https://gdpr-info.eu/art-18-gdpr/?utm_source=chatgpt.com "Art. 18 GDPR – Right to restriction of processing"), [EDPB](https://www.edpb.europa.eu/sme-data-protection-guide/respect-individuals-rights_en "Respect individuals’ rights \| European Data Protection Board"))|
|**Rate‑Limit & Logging**|• Nginx‑Rate‑Limit `10r/m`. • Alle Requests in `rights_api.log` inkl. IP, UserAgent.|

---

## 6  Datenminimierung & Security

|Aufgabe|Konkret zu erledigen|
|---|---|
|**Minimierungs‑Policy**|Dokument „Data Retention & Minimisation“: definiert Spalten‑Level‑Retention (z. B. Logs 90 Tage, Metrics 13 Monate) ([GDPR](https://gdpr-info.eu/art-5-gdpr/?utm_source=chatgpt.com "Art. 5 GDPR – Principles relating to processing of personal data"), [Homepage \| Data Protection Commission](https://www.dataprotection.ie/en/individuals/data-protection-basics/principles-data-protection?utm_source=chatgpt.com "Principles of Data Protection"))|
|**Automatisierte Purges**|Cron mit MongoDB‑`deleteMany({createdAt:{$lt:…}})` ; alle Jobs in `/scripts/purge_*.php` protokollieren Logs.|
|**TOMs nach Art. 32**|• AES‑256‐Encryption‐at‑Rest, TLS 1.3 in Transit • RBAC (least privilege) • Off‑site Backups • Jährlicher Pen‑Test • Vierteljährliche Restore‑Übung ([GDPR](https://gdpr-info.eu/art-32-gdpr/?utm_source=chatgpt.com "Art. 32 GDPR – Security of processing - General Data Protection ..."), [GDPR](https://gdpr-info.eu/issues/encryption/?utm_source=chatgpt.com "Encryption - General Data Protection Regulation (GDPR)"))|

---

## 7  Auftragsverarbeitung (Art. 28)

|Aufgabe|Konkret zu erledigen|
|---|---|
|**Vertragsregister**|Ordner `/contracts/processors/` mit allen DPAs und Sub‑Processors. Tabelle enthält Gegenstand, Dauer, Art der Daten, TOM‑Verweis ([GDPR](https://gdpr-info.eu/art-28-gdpr/?utm_source=chatgpt.com "Art. 28 GDPR – Processor - General Data Protection Regulation ..."))|
|**Onboarding‑Checkliste**|Enthält Security‑Questionnaire, DPA‑Status, Drittlandtransfer‑SCCs.|
|**Sub‑Processor Monitoring**|RSS‑Watch auf Ankündigungen; bei Änderung 30‑Tage‑Opt‑Out.|

---

## 8  Breach‑Process (Art. 33)

|Aufgabe|Konkret zu erledigen|
|---|---|
|**Incident‑Runbook**|Markdown‐Dokument mit 1) Definition Breach, 2) T‑0: Isolieren, 3) T+4h: DPO‑Call, 4) T+24h: Faktenblatt, 5) T+72h: Meldung SA + Betroffene. ([GDPR](https://gdpr-info.eu/art-33-gdpr/?utm_source=chatgpt.com "Notification of a personal data breach to the supervisory authority"))|
|**Breach‑Register**|Mongo‑Collection `breaches` mit Feldern `{id, detectedAt, notifiedSA, notifiedUsers, rootCause, lessonsLearned}`.|
|**Table‑Top‑Übung**|Halbjährliche Simulation; Ergebnis‑PDF in `/breach_drills/`.|

---

### Nächste Schritte

1. **RoPA‑Excel‑Template ausfüllen** und direkt ins Repo committen.
    
2. **Mapping‑YAML** ins Config‑Verzeichnis legen und beim nächsten Feature‑Sprint verpflichtend abfragen.
    
3. **Privacy‑Notice‑Abschnitt „Rechte ausüben“** noch heute deployen, damit die Rechte‑Endpoints einen sichtbaren Einstiegspunkt haben.
    

Damit ist jedes To‑do mit einer klaren Deliverable, einem Speicherort und, falls nötig, einem Code‑Task hinterlegt – Sie können die Punkte nun direkt einem Projektboard oder Sprint zuweisen.