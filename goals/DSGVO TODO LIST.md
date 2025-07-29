| Phase                                                                                                         | Why it comes **now**                                                                                                                    | Key Deliverables                                                                                                                                                                                           |
| ------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1️⃣ [[Data Inventory & Lawful Basis]]                                                                       | Every later item (Privacy Notice, RoPA, retention, DPIA, even API permissions) needs to know **_what_** data you process and **_why_**. | * • Mini-workshop: list every table/collection, field, API, log*• **Rechtsgrundlagen-Matrix** (Art. 6 bases + purposes)• Draft of RoPA columns (“activity, purpose, lawful basis, retention, recipients…”) |
| **2️⃣ RoPA skeleton**                                                                                         | Once you have inventory + lawful basis you can fill 80 % of the RoPA.                                                                   | _Markdown / sheet with one line per processing activity_                                                                                                                                                   |
| 3️⃣ [[Privacy Policy Notice]]                                                                                 | Needs the same info you just wrote down (purposes, rights, contact). Finishing it early unblocks Design / App-store listing.            | _Public-facing notice + “exercise your rights” instructions_                                                                                                                                               |
| **4️⃣ [[Consent Management logic]] <br>- [ ] fehlt nur noch das die consent flags wirklich effektiv alle sind | Requires: list of optional processing AND wording from Privacy Policy.                                                                  | _Frontend toggles ↔ backend flags wired & tested_                                                                                                                                                          |
| **5️⃣ User-Rights API** (rectification, delete, export, restriction, objection)                               | Can now rely on correct consents + documented data locations.                                                                           | _Endpoints + [[rate-limiting]] + 2-admin rule_                                                                                                                                                             |
| **6️⃣ Retention & Purge scripts**                                                                             | Only possible after data map (Phase 1) and RoPA (Phase 2) specify how long to keep each data set.                                       | _Cron / Lambda cleaners, log purge rotation_                                                                                                                                                               |
| **7️⃣ DPIA / Security / Breach Plan**                                                                         | Uses outputs of earlier phases (risk-scoring, safeguards).                                                                              | _Risk matrix, technical & organisational measures, breach SOP_                                                                                                                                             |
| **8️⃣ Sub-Processors & DPAs**                                                                                 | Easy once you know which services you actually use (inventory) and have legal basis & retention decided.                                | _Signed agreements stored, list inserted into RoPA & Privacy Policy_                                                                                                                                       |
| **9️⃣ Training / Review Cycle**                                                                               | Final housekeeping when everything else exists.                                                                                         | _On-boarding doc, quarterly calendar reminder_                                                                                                                                                             |



## short form todo list
- [ ] **[[DSGVO TODO MASTERLIST]]**
  - [ ] **[[RoPA – Verarbeitungsverzeichnis]]**
    - [ ] Dokument als Markdown / Spreadsheet anlegen  
    - [ ] Prüfen, ob *jede* Verarbeitungstätigkeit (Profil-Bilder, Standort, Voice-Memos, Chat-Logs, Crash-Logs …) enthalten ist  
    - [ ] Sicherstellen, dass für alle Einwilligungs-Fälle eine Rechtsgrundlage hinterlegt ist  
    - [ ] Versionierung & Change-Log pflegen  
  - [ ] **[[Rechtsgrundlagen-Matrix (Art. 6 DSGVO)]]**
    - [ ] JSON-/YAML-Datei mit Feld → Rechtsgrundlage → Zweck  
    - [ ] Double-check: optionale Newsletter, Crashlytics, Analytics → Einwilligung  
  - [ ] **[[Privacy Policy & Impressum]]**
    - [ ] Abschnitt „So übst du deine Rechte aus“ (inkl. Restriction-Mail)  
    - [ ] Link dauerhaft in App-Footer / Settings  
  - [ ] **[[Consent Management]]**
    - [x] In-App Permissions Page  
    - [ ] Prüfen, ob `location`-Consent wirklich Feature toggelt  
    - [ ] Logische Verknüpfung Frontend ↔ Backend-Flags  
  - [ ] **[[User-Rights-API]]**
    - [x] `delete_me.php`  
    - [x] Cron `cleanup_delete_requests.php` (30 d)  
    - [x] `request_me.php` (+24 h Export Cleaner)  
    - [x] `update_userdata.php` (Rectification)  
    - [x] `restriction`/`unrestrict` Endpoints + separate `restricted_users` Coll.  
    - [ ] **Rate-Limiting** & Abuse-Protection (per IP / UID)  
    - [ ] 2-Admin-Rule für manuelle Admin-Löschungen  
  - [ ] **[[Breach Response Plan]]**
    - [ ] 72-h Meldefrist Ablaufplan  
    - [ ] Interne & Behörde-Kontaktliste  
  - [ ] **[[Data Retention & Erasure Policy]]**
    - [ ] Mapping: Datenkategorie → Löschfrist / Anonymisierung  
    - [ ] Script → Autopurge old chat voice-files, logs …  
  - [ ] **[[Sub-Processors & DPAs]]**
    - [ ] Liste Hosting, Mailgun, Firebase Crashlytics …  
    - [ ] DPAs digital ablegen & verknüpfen  
  - [ ] **[[DPIA (check if needed)]]**
    - [ ] Risiko-Scoring Voice, Location, Sensitive Profiles  
  - [ ] **[[Security Policy]]**
    - [ ] Encryption at rest / in transit  
    - [ ] Access-Control, Audit-Logs, 2-factor for Admin  
  - [ ] **[[Training & Awareness]]**
    - [ ] Onboarding-Sheet für new devs → GDPR basics  
  - [ ] **[[RoPA / Policy Review Cycle]]**
    - [ ] Quartals-Reminder – task in Jira/CRON  




## longform

To get fully GDPR-compliant, you’ll need both the user-facing notices and a suite of internal records and policies. Here’s the short checklist:

---

## 1. User-Facing Documents

1. **Privacy Notice (“Privacy Policy”)**
    
    - Explains what you collect, why (Art. 6), legal bases, retention, recipients, rights (rectification, erasure, restriction, objection, portability), and how to exercise them.
        
    - Should live on your website/app (“Einstellungen → Datenschutz” or a footer link).
        
2. **Terms & Conditions**
    
    - Your service contract (Art. 6 (1)(b)). References the Privacy Policy and user obligations.
        
3. **Cookie & Tracking Policy**
    
    - If you use cookies, analytics, third-party trackers.
        
4. **Consent Dialogs**
    
    - In-app screens for AGB/Privacy acceptance, newsletter opt-in, sensitive data (religion/politics), OS-permissions—all documented and timestamped.
        

---

## 2. Internal Records & Policies

1. **RoPA (Record of Processing Activities)**
    
    - A living register of every processing operation: what data, why, categories, recipients, retention.
        
2. **Data Processing Agreements (DPAs)**
    
    - With any subprocessors (e.g. hosting, email, analytics).
        
3. **Data Protection Impact Assessment (DPIA)**
    
    - For any high-risk processing (e.g. voice memos, location matching).
        
4. **Data Retention & Erasure Policy**
    
    - Defines how long you keep each category and routines (e.g. 30-day deletion queue).
        
5. **Breach Response Plan**
    
    - Procedure and contacts for detecting, reporting, and notifying breaches within 72 h.
        
6. **Internal Privacy & Security Policy**
    
    - Who has access, roles, encryption, password rules, 2-man rule for admin deletions, rate-limiting, audit logs.
        
7. **Records of Consent / Withdrawal**
    
    - An auditable log of every time a user gives/withdraws consent (newsletter, sensitive profile fields).
        

---

## 3. User-Rights API & Documentation

- **Endpoints**:
    
    - `delete_me.php` (Art. 17)
        
    - `request_me.php` (Art. 15)
        
    - `delete_me` cron (30 days later)
        
    - `restrict_me.php` / `unrestrict_me.php` (Art. 18)
        
    - `get_consents.php` / `consent.php` (Art. 21)
        
    - `update_userdata.php` (Art. 16)
        
- **Internal API Spec**:
    
    - Document each endpoint, parameters, security (JWT + re-auth), rate limits, retention of logs.
        

---

## 4. Other Helpful Docs

- **Vendor & Sub-processor List**
    
- **Third-Country Transfer Mechanisms** (if any)
    
- **Audit & Training Records** for staff
    
- **DPIA Review Schedule**
    
- **Data Subject Request Playbook**
    

---

With these in place, you’ll have both the _user-facing notices_ they need to see, _internal policies_ you need to run by, and _records_ to prove compliance in an audit.