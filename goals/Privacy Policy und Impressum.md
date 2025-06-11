Below you’ll find three things you can copy straight into your repo:

1. **Two ready-to-paste Markdown files**
    
    - `privacy_policy.md` – DSGVO-konform, deutsch/englisch gemischt
        
    - `impressum.md` – § 5 TMG / § 18 MStV
        
2. **Flutter & PHP glue code** for the in-app links
    
3. **Google Play “Data-Safety” cheat sheet** (what to tick in the Console)
    

_Everything is kept minimal; drop in your real company data and tweak wording as needed. This is _not_ formal legal advice._

1 privacy_policy.md (Example)
```
# Datenschutzerklärung / Privacy Policy

_Last updated: 11 June 2025_

## 1. Verantwortlicher / Data Controller  
MyApp UG (haftungsbeschränkt)  
Musterstraße 1, 12345 Berlin, Germany  
privacy@myapp.example

## 2. Zweck & Rechtsgrundlagen (Art. 6 DSGVO)

| Zweck                                     | Datenkategorien                                      | Rechtsgrundlage |
|-------------------------------------------|------------------------------------------------------|-----------------|
| Betrieb Ihres Nutzerkontos & Chats        | username, E-Mail, Passwort-Hash, Chat-Nachrichten    | Vertrag Art. 6 (1)(b) |
| Sicherheits- & Betrugsprävention          | verkürzte IP, Fingerprint-Hash, Verification Codes   | Berechtigtes Interesse Art. 6 (1)(f) |
| Crash- & Nutzungsstatistik (optional)     | pseudonyme IDs, Geräte-Infos                         | Einwilligung Art. 6 (1)(a) |
| Newsletter (optional)                     | E-Mail, Öffnungs-/Klick-Events                       | Einwilligung Art. 6 (1)(a) |
| Buchhaltung / Steuer                      | Name, Adresse, Stripe-ID (bei Abo)                   | Gesetzliche Pflicht Art. 6 (1)(c) |

## 3. Weitergabe an Dritte  
AWS eu-central-1 (Hosting), Google Ireland Ltd. (Firebase, _nur bei Einwilligung_),  
SendGrid (Newsletter, _nur bei Einwilligung_).  
**Kein** Transfer in Drittstaaten ohne angemessenen Schutzmechanismus (SCC).

## 4. Speicherdauer  
Wir löschen Account-Daten spätestens 30 Tage nach Kontolöschung,  
Rechnungsdaten nach 10 Jahren (§ 147 AO).

## 5. Ihre Rechte  
Auskunft, Berichtigung, Löschung, Datenportabilität, Widerspruch, Beschwerde bei der Berliner Aufsichtsbehörde.

## 6. Einwilligung & Widerruf  
Analyse- und Newsletter-Tracking erfolgen **nur** nach aktiver Zustimmung.  
Ein Widerruf ist jederzeit in der App unter _Einstellungen → Datenschutz_ möglich.

## 7. Sicherheit  
TLS 1.3, Passwort-Hashing mit Argon2id, Zugriff nur per IAM-Roles.

## 8. Änderungen  
Diese Erklärung wird bei Funktions- oder Gesetzesänderungen aktualisiert.

```

2 impressum.md
```
# Impressum

Angaben gemäß § 5 TMG / § 18 MStV  

**MyApp UG (haftungsbeschränkt)**  
Musterstraße 1  
12345 Berlin  
Deutschland

Handelsregister: HRB 123456 (Berlin Charlottenburg)  
Geschäftsführer: Max Mustermann  
USt-IdNr.: DE123456789

Kontakt:  
Tel.: +49 30 1234567  
E-Mail: info@myapp.example

Verantwortlich für den Inhalt nach § 18 Abs. 2 MStV:  
Max Mustermann, Anschrift wie oben.

EU-Streitschlichtung: https://ec.europa.eu/consumers/odr  
Wir sind weder verpflichtet noch bereit, an Streitbeilegungsverfahren vor einer Verbraucherschlichtungsstelle teilzunehmen.

```

---

## 3 Flutter – Link in Onboarding
```
// Inside your onboarding screen
RichText(
  text: TextSpan(
    style: Theme.of(context).textTheme.bodySmall,
    children: [
      const TextSpan(text: 'Durch die Registrierung akzeptierst du unsere '),
      TextSpan(
        text: 'Datenschutzerklärung',
        style: const TextStyle(decoration: TextDecoration.underline),
        recognizer: TapGestureRecognizer()
          ..onTap = () => launchUrl(Uri.parse('https://myapp.example/privacy')),
      ),
      const TextSpan(text: ' und das '),
      TextSpan(
        text: 'Impressum',
        style: const TextStyle(decoration: TextDecoration.underline),
        recognizer: TapGestureRecognizer()
          ..onTap = () => launchUrl(Uri.parse('https://myapp.example/impressum')),
      ),
      const TextSpan(text: '.'),
    ],
  ),
);

```

4 PHP route (static file)
```
// routes.php
$app->get('/privacy', fn() => readfile(__DIR__.'/legal/privacy_policy.md'));
$app->get('/impressum', fn() => readfile(__DIR__.'/legal/impressum.md'));
```

5 Google Play “Data-Safety” quick answers
### 5 · Google Play “Data-Safety” quick answers  (example for your app)

| Question in the Play - Console | Your answer | Notes / why |
|--------------------------------|-------------|-------------|
| Does the app **collect** or **share** personal data? | **Yes – collect** | The moment you create a user profile, you “collect”. |
| Is all user data **encrypted in transit**? | **Yes** | Your API is behind HTTPS/TLS 1.3. |
| Do you provide a way to **request deletion**? | **Yes** | Settings → Privacy → Delete account (GDPR Art. 17). |
| Data **shared** with third parties | **None** | Processors (AWS, SendGrid, Firebase) count as “collected”, not “shared”. |
| Types of data **collected** | **Personal Info → Name, E-mail**<br>**Messages**<br>**Identifiers → User ID**<br>**App Activity (optional)** | Tick “Analytics” only if user gave consent. |
| Purpose of collection | **App functionality** (all mandatory fields)<br>**Analytics** (optional) | Match what you describe in the Privacy Policy. |
| Data deletion policy | “Users can delete their account in-app; personal data is erased from production within 30 days, backups after 30 days more.” | This text must match the real retention logic. |

> **Tip:** Keep a screenshot of every answer you give. If Google flags the listing later, you can prove what you declared.

---

## 6 · Checklist for submitting the Privacy-Policy & Data-Safety info

|Step|Where|Action|
|---|---|---|
|1|**App content → Privacy Policy**|Paste `https://myapp.example/privacy`|
|2|**App content → Data-safety**|Walk through the wizard using the table above|
|3|**Store listing → Contact details**|Add the Impressum URL in “Website”|
|4|**App releases → Testing → Production testing**|Upload the AAB + sign Data-safety form|
|5|**Publishing overview**|Fix any “Policy issue” warnings before requesting review|

Google usually reviews the **Data-Safety form** together with your first public release. First-time review can take 3-7 days.

---

## 7 · Generating HTML from your Markdown files (optional)

If you want to auto-publish the Markdown files as static HTML on deploy, a one-liner in your CI works:

```bash
# convert.sh
pandoc legal/privacy_policy.md -o public/privacy.html --metadata title="Privacy Policy"
pandoc legal/impressum.md       -o public/impressum.html --metadata title="Impressum"
```

Add `./convert.sh` to your build pipeline (GitHub Actions, GitLab CI, etc.), commit the generated `public/*.html`, and point the Play-Store URLs at the HTML versions. Google doesn’t care whether the file ends in `.html`, `.md`, or no extension—as long as it responds with plain text/HTML.

---

## 8 · Release “safety belt” before going live

1. **Internal testing track (max 100 users)** – sanity-check consent flow.
    
2. **Closed testing / Production testing** – add 500–2 000 selected users; counts as “published” for investors.
    
3. **Rollout by percentage** – in Production, start at 5 % of countries ⇒ watch Crashlytics & ANRs.
    
4. **Legal spot-check** – once the first real users delete their account, verify the purge job ran.
    

---
