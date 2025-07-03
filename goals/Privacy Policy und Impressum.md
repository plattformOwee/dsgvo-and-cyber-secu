Below is an **updated bundle** that matches your **new Mongo-structure** (religion / politics bubbles, location radius, consent flags, etc.).  
Just drop the Markdown files into `/legal/`, adjust the placeholders (🟡) and you’re good to commit.

---

## 1 · `privacy_policy.md` _(template)_

```markdown
# Datenschutzerklärung / Privacy Policy

_Last updated: 1 July 2025_

## 1. Verantwortlicher / Data Controller  
🟡 YourApp UG (haftungsbeschränkt)  
🟡 Musterstraße 1, 12345 Berlin, Germany  
privacy@yourapp.example

## 2. Zwecke & Rechtsgrundlagen

| Zweck / Activity | Datenkategorien | Rechtsgrundlage |
|------------------|-----------------|-----------------|
| **Betrieb Ihres Nutzerkontos & Chats** | Name, E-Mail, Passwort-Hash, Profilbilder, Chat-Nachrichten, Eisbrecher-Antworten | Vertrag Art. 6 (1)(b) |
| **Freiwillige Profilangaben** (Religion, politische Ansicht) | Religion, Politik | Einwilligung Art. 6 (1)(a) i. V. m. Art. 9 (2)(a) |
| **Standort-basiertes Matching** (optional) | Geokoordinaten (Breite/Länge), Radius | Einwilligung Art. 6 (1)(a) |
| **Sicherheits- & Betrugsprävention** | verkürzte IP, Fingerprint-Hash, Verification-Tokens | Berechtigtes Interesse Art. 6 (1)(f) |
| **Crash- & Nutzungsstatistik** (optional) | pseudonyme Geräte-ID, Stack-Trace, App-Events | Einwilligung Art. 6 (1)(a) |
| **Newsletter / Produkt-Mails** (optional) | E-Mail, Öffnungs- / Klick-Events | Einwilligung Art. 6 (1)(a) |
| **Buchhaltung & Steuer** | Name, Adresse, Stripe-ID (bei Abo) | Gesetzliche Pflicht Art. 6 (1)(c) |

## 3. Weitergabe an Dritte  
AWS (EU-Central, Hosting), Google Ireland Ltd. (Firebase Crashlytics _bei Einwilligung_),  
SendGrid (Newsletter _bei Einwilligung_), Stripe (Payment, gemeinsamer Verantwortlicher).  
**Kein** Drittland-Transfer ohne geeignete Garantien (Standard­vertrags­klauseln).

## 4. Speicherdauer  
Profil- & Chat-Daten: Löschung spätestens 30 Tage nach Account-Löschung.  
Rechnungs- / Steuerdaten: 10 Jahre (§ 147 AO).

## 5. Ihre Rechte  
Auskunft, Berichtigung, Löschung, Daten­portabilität, Widerruf, Beschwerde bei der Berliner Aufsichts­behörde.

## 6. Einwilligung & Widerruf  
Standort, sensible Profilfelder, Analyse- und Newsletter-Tracking verarbeiten wir **nur** nach aktiver Zustimmung.  
Widerruf jederzeit in _Einstellungen → Datenschutz_.

## 7. Sicherheit  
TLS 1.3, Passwort-Hash Argon2id + Salt, Verschlüsselung-at-Rest, Zugriff nur via IAM-Rollen.

## 8. Änderungen  
Diese Erklärung wird bei Funktions- oder Gesetzes­änderungen aktualisiert.
```

---

## 2 · `impressum.md` _(unchanged except for placeholders)_

```markdown
# Impressum

Angaben gemäß § 5 TMG / § 18 MStV  

**🟡 YourApp UG (haftungsbeschränkt)**  
🟡 Musterstraße 1  
12345 Berlin  
Deutschland

Handelsregister: HRB 🟡 (Bezirksgericht Berlin-Charlottenburg)  
Geschäftsführer: 🟡 Max Mustermann  
USt-IdNr.: 🟡 DE …  

Kontakt:  
Tel.: +49 30 🟡  
E-Mail: info@yourapp.example

Verantwortlich i.S.d. § 18 Abs. 2 MStV:  
🟡 Max Mustermann, Anschrift wie oben.

EU-Streitbeilegung: <https://ec.europa.eu/consumers/odr>  
Wir nehmen nicht an Streitbeilegungs­verfahren vor einer Verbraucher­schlichtungs­stelle teil.
```

---

## 3 · Flutter onboarding links

_(URLs bleiben gleich; Datei-Namen änderten sich nicht)_

```dart
// ... inside onboarding screen
TextSpan(
  text: 'Datenschutzerklärung',
  recognizer: TapGestureRecognizer()
    ..onTap = () => launchUrl(Uri.parse('https://yourapp.example/privacy')),
),
...
```

---

## 4 · PHP routes (static)

```php
$app->get('/privacy', fn() => readfile(__DIR__.'/legal/privacy_policy.md'));
$app->get('/impressum', fn() => readfile(__DIR__.'/legal/impressum.md'));
```

---

## 5 · Google Play “Data-Safety” – updated cheat-sheet

|Console question|Your answer|Notes|
|---|---|---|
|Does the app collect / share personal data?|**Yes – collect**|Profile & chat data.|
|All data encrypted in transit?|**Yes**|TLS 1.3.|
|Deletion request available?|**Yes**|Settings → Privacy → Delete Account.|
|Data **shared** with third parties|**None**|AWS, SendGrid etc. = processors (≠ shared).|
|**Collected** data types|• Personal Info → Name, E-mail• Messages• Identifiers → User ID• **Sensitive Info → Religion / Political Beliefs** (opt-in)• **Location** (opt-in)• App Activity (Analytics, opt-in)|Tick only optional items if user gave consent.|
|Purpose|**App functionality** (mandatory)**Analytics** (optional)|Keep wording consistent with policy.|
|Data-deletion policy|“Users can delete account in-app; data purged from prod ≤ 30 days, backups ≤ 60 days.”|Match real retention logic.|

_Take a screenshot of each answer for audit trails._

---

## 6 · Submission checklist (unchanged)

1. **App content → Privacy Policy** – paste `https://yourapp.example/privacy`
    
2. **App content → Data-safety** – fill wizard with table above
    
3. **Store listing → Contact details** – set Impressum URL
    
4. **Releases → Testing → Production testing** – upload AAB, sign form
    
5. Resolve any “Policy issue” before review
    

---

## 7 · Markdown → HTML one-liner (CI)

```bash
pandoc legal/privacy_policy.md -o public/privacy.html   --metadata title="Privacy Policy"
pandoc legal/impressum.md       -o public/impressum.html --metadata title="Impressum"
```

---

## 8 · “Safety belt” rollout (same steps)

1. Internal track (≤ 100) → consent flow sanity-check
    
2. Closed / Production testing (500–2 000)
    
3. Gradual 5 % rollout, monitor crashes & ANRs
    
4. Legal spot-check: first real deletion ⇒ verify purge worker
    

---

**That’s it – all pieces now reflect your current schema (sensitive bubbles, geo-matching, consent flags).**  
If you want a ready-made ZIP with both Markdown files or more Play-Console screenshots, just holler!