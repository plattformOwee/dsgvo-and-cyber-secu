# new version
### RoPA / Verarbeitungsverzeichnis – Collection **users** (v 2025-07-01)

| Field / Gruppe                                               | Mongo Path                                          | Kategorie (Art. 4 Nr.)                | Zweck / Verarbeitungstätigkeit          | Rechtsgrundlage*                                      | Lösch-/Aufbewahrungsfrist                                |
| ------------------------------------------------------------ | --------------------------------------------------- | ------------------------------------- | --------------------------------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **_id**                                                      | users.\_id                                          | Identifier                            | Primärschlüssel für Datensatzverwaltung | Vertrag Art. 6 (1)(b)                                 | Account-Löschung + 30 Tage (Backup)                      |
| **Basis-Profil** – firstName, lastName, name                 | profile.topSection.\*                               | Personenbez. Daten                    | Anzeige in Profil & Match­listen        | Vertrag Art. 6 (1)(b)                                 | Account-Löschung + 30 Tage                               |
| **Profilbilder**                                             | profile.topSection.profileImages[]                  | Bilddaten                             | Visuelle Darstellung im Matching / Chat | Vertrag Art. 6 (1)(b)                                 | Bis Entfernen durch Nutzer oder Account-Löschung         |
| **InfoBubbles.gender, languages**                            | profile.topSection.infoBubbles.gender / languages[] | Personenbez. Daten                    | Filter- & Profilanzeige                 | Vertrag Art. 6 (1)(b)                                 | Account-Löschung + 30 Tage                               |
| **InfoBubbles.religion, politics**                           | profile.topSection.infoBubbles.religion / politics  | *Bes. Kat.* (Art. 9 – Weltanschauung) | Freiwillige Profilangabe                | **Einwilligung Art. 9 (2)(a) i. V. m. Art. 6 (1)(a)** | Widerruf oder Account-Löschung                           |
| **age**                                                      | profile.topSection.age                              | Personenbez. Daten                    | Altersfilter / Jugendschutz             | Vertrag Art. 6 (1)(b)                                 | Account-Löschung + 30 Tage                               |
| **Icebreaker & Fragen** – question/answer                    | profile.elements[].content                          | Nutzergenerierte Inhalte              | Persönliche Selbstdarstellung           | Vertrag Art. 6 (1)(b)                                 | Nutzer löschbar; sonst Account-Löschung + 30 Tage        |
| **Suchpräferenzen** – genders, ageRange, religion, politics  | search_filter.searching_for.\*                      | Präferenzdaten                        | Match-Algorithmen                       | Vertrag Art. 6 (1)(b)                                 | Aktualisierung bei Änderung; Account-Löschung + 30 Tage  |
| **Standort & Radius**                                        | search_filter.location_radius.location / radius     | Standortdaten (Geodaten)              | Nähe-Matching                           | Einwilligung Art. 6 (1)(a)                            | Widerruf oder Account-Löschung                           |
| **E-Mail**                                                   | userdata.email                                      | Kontaktdaten                          | Kommunikation, Passwort-Reset           | Vertrag Art. 6 (1)(b)                                 | Account-Löschung + 30 Tage                               |
| **Verification-Code / -Expiry**                              | userdata.verification_code(_expiry)                 | Sicherheits-Token                     | Konto-Verifikation / Betrugsprävention  | Berecht. Interesse Art. 6 (1)(f)                      | Auto-Löschung nach Erfolg oder 48 h                      |
| **is_verified**                                              | userdata.is_verified                                | Statusflag                            | Funktionsfreischaltung                  | Vertrag Art. 6 (1)(b)                                 | Account-Löschung + 30 Tage                               |
| **fingerprint_hash**                                         | userdata.fingerprint_hash                           | Geräte-Fingerprint (pseudonym)        | Missbrauchserkennung / Session-Security | Berecht. Interesse Art. 6 (1)(f)                      | Account-Löschung + 30 Tage                               |
| **Chat-Liste** – receiver_id, receiver, last_message, unread | chats.{chatId}.\*                                   | Kommunikations-Metadaten              | Gesprächsverwaltung                     | Vertrag Art. 6 (1)(b)                                 | Chat gelöscht vom Nutzer oder Account-Löschung + 30 Tage |
| **Profil-URL (Chat-Avatar)**                                 | chats.{chatId}.profileURL                           | Bilddaten                             | Anzeige Chat-Übersicht                  | Vertrag Art. 6 (1)(b)                                 | Wie Chat-Liste                                           |
| **Likes-Liste**                                              | likes[]                                             | Beziehungs­daten                      | Match-Logik                             | Vertrag Art. 6 (1)(b)                                 | Entfernen durch Nutzer oder Account-Löschung + 30 Tage   |

\* **Rechtsgrundlage-Legende**

- **Vertrag**: Art. 6 (1)(b) DSGVO – Erfüllung Nutzungsvertrag  
- **Einwilligung**: Art. 6 (1)(a) DSGVO / Art. 9 (2)(a) DSGVO (besondere Daten)  
- **Berechtigtes Interesse**: Art. 6 (1)(f) DSGVO – Security / Fraud

> **TOM-Spalte** (separate Sheet/Tab): TLS 1.3, Argon2id + Salt, Mongo Auth + TLS, VPC-Firewall, Role-Based-Access, Encryption-at-Rest.












# version old
PUK  mnet: 37910620

| Field                      | Mongo Path                                 | Category                  | Purpose                                | Legal Basis                             | Retention                                 |
| -------------------------- | ------------------------------------------ | ------------------------- | -------------------------------------- | --------------------------------------- | ----------------------------------------- |
| _id                        | users._id                                  | Identifier                | Primary key for user record management | Contract&nbsp;(Art.&nbsp;6 (1) b)       | Until account deletion + 30 days (backup) |
| username                   | users.user_data.username                   | Personal identifier       | Display name & login (if applicable)   | Contract&nbsp;(Art.&nbsp;6 (1) b)       | Until account deletion + 30 days          |
| firstname                  | users.user_data.firstname                  | Personal data             | Personalisation / account profile      | Contract&nbsp;(Art.&nbsp;6 (1) b)       | Until account deletion + 30 days          |
| lastname                   | users.user_data.lastname                   | Personal data             | Personalisation / account profile      | Contract&nbsp;(Art.&nbsp;6 (1) b)       | Until account deletion + 30 days          |
| email                      | users.user_data.email                      | Contact data              | Account communication, password reset  | Contract&nbsp;(Art.&nbsp;6 (1) b)       | Until account deletion + 30 days          |
| password_hash              | users.user_data.password_hash              | Authentication credential | Secure login authentication            | Contract&nbsp;(Art.&nbsp;6 (1) b)       | Until account deletion + 30 days          |
| created_at                 | users.user_data.created_at                 | Metadata                  | Audit trail / troubleshooting          | Legitimate interest (Art.&nbsp;6 (1) f) | Until account deletion + 30 days          |
| verification_code & expiry | users.user_data.verification_code(_expiry) | Security token            | E-mail verification / fraud prevention | Legitimate interest (Art.               |                                           |
