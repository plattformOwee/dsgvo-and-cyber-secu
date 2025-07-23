# neue aufgaben 
grant: 
### 3.1 Rectification endpoint (`rectify_me.php`)
> - ersetzt durch datenänderung aber wir ändern die info page, so das sie einen edit und nicht edit mode hat mit edit symbol das dann die methode abfragt und dann muste oder was auch immer und dann erst kann man die eigenen daten ändern. Gleiches wird ein malig beim ersten mal drücken irgendeines eingabefeldes ausgelößt.

### 3.2 Restriction endpoint (`restrict_me.php`)
- **Authenticate** the user.
- **Set** a flag, e.g. `processing_restricted_at: new BSON\UTCDateTime()`, on their MongoDB document.
- **Ensure** your business logic checks this flag and suspends all non-essential processing.
- **Return** `200 OK` with the restriction timestamp.  
    [GDPR](https://gdpr-info.eu/art-18-gdpr/?utm_source=chatgpt.com)[GDPR.eu](https://gdpr.eu/article-18-right-to-restriction-of-processing/?utm_source=chatgpt.com)
> 1. ein button erstellen in "Datenschutz und cyber securtiy" page der erscheint wann auch immer ein prozess läuft, dazu ein par flags einführen also `processing_restricted_at: new BSON\UTCDateTime()`, 
> 2. die flag kann man am besten auch per endpoint als mitarbeiter setzen oder so
> 3. jeden endpoint darauf reagieren lassen vielleicht kann man das in bootstrap oder irgendwo integrieren damit nicht jedes script geändert werden muss

### 3.3 Objection endpoint (`object_me.php`)
- **Authenticate** the user.
    
- **Accept** a parameter indicating the purpose (e.g. `"marketing"`, `"profiling"`).
    
- **Store** a sub-field like `objections.marketing = new BSON\UTCDateTime()`.
    
- **Adjust** downstream services (e.g. mailer) to skip any processing for that purpose.
    
- **Return** `200 OK` with the purpose objected and timestamp.



# new
## Praxis-Mapping Art. 6 / Art. 9 DSGVO – **neues Datenmodell**

| Verarbeitungsvorgang                                    | Betroffene Datenfelder*                                            | Rechtsgrundlage (Art. 6 / 9)                     | WIE du sie erfüllst (UX / Dokument)                                                | Was im Code / DB landet                            |
| ------------------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------ | ---------------------------------------------------------------------------------- | -------------------------------------------------- |
| **Kern-Account & Chat**                                 | `profile.topSection.name`, `userdata.email`, Chat-Nachrichten      | **lit. b** (Vertrag)                             | ToS + Privacy-Checkbox im Onboarding                                               | kein Consent-Flag nötig                            |
| **Profilbilder & Eisbrecher / Fragen**                  | `profile.profileImages[]`, `profile.elements[].content.answer`     | **lit. b**                                       | Upload-Button bzw. Freitextfelder → Teil der Kern­funktionen                       | –                                                  |
| **Freiwillige „InfoBubbles“ – Religion, Politik**       | `infoBubbles.religion`, `infoBubbles.politics`                     | **Art. 9 (2)(a)** + **Art. 6 (1)(a)** (Einwill.) | Extra-Schalter „Diese sensiblen Angaben veröffentlichen“ (Opt-in, default **off**) | `consents.profile_sensitives.granted = true/false` |
| **Standort + Radius** (Nah-Matching)                    | `search_filter.location_radius.location / radius`                  | **lit. a** (Einwill.)                            | OS-Location-Prompt + App-Dialog „Standort­basiertes Matching aktivieren?“          | `consents.location.granted = true/false`           |
| **Device- & Abuse-Logs**                                | IP-Logs (Server), `userdata.fingerprint_hash`, `verification_code` | **lit. f** (berechtigtes Interesse)              | Absatz „Sicherheit“ in Privacy-Policy + interner Balancing-Test                    | kein Opt-in; Rotations-/Löschfristen               |
| **Crash- & Nutzungsanalyse** (Firebase / Sentry)        | pseudonyme Instanz-ID, Stacktrace, Device-Infos                    | **lit. a** *oder* **lit. f***                    | Vor 1. Event: Consent-Dialog-Checkbox **„App-Statistiken senden“**                 | `consents.analytics.granted` + timestamp           |
| **Marketing / Newsletter**                              | `userdata.email`, Öffnungs-Tracking                                | **lit. a**                                       | Getrennte Checkbox (abgewählt), Double-Opt-In-Mail                                 | `consents.newsletter.granted` + timestamp          |
| **Zahlungs- / Rechnungsdaten** (In-App-Abo außer Store) | Name, Adresse, Stripe-ID, Transaktions-ID                          | **lit. c** (gesetzl. Pflicht)                    | Verweis auf HGB/AO im Privacy-Policy; Datensatz in `finance`-Collection            | 10 J. Aufbewahrung                                 |

\* **Synonyme Felder** (firstName/lastName) sind in „Kern-Account“.

\*\* Viele Start-ups deklarieren Crash-Logs unter **lit. f**; dann Balancing-Test + Opt-out nötig.

---

### 1 · Flutter-UI – erweiterter Consent-Dialog

```dart
class ConsentDialog extends StatefulWidget {
  final void Function({
    required bool analytics,
    required bool newsletter,
    required bool sensitives,
    required bool location,
  }) onSave;
  const ConsentDialog({required this.onSave, super.key});

  @override
  State<ConsentDialog> createState() => _ConsentDialogState();
}

class _ConsentDialogState extends State<ConsentDialog> {
  bool analytics = false;
  bool newsletter = false;
  bool sensitives = false;      // Religion/Politik freigeben
  bool location  = false;      // Standort-Matching

  @override
  Widget build(BuildContext context) {
    return AlertDialog(
      title: const Text('Datenschutz-Einstellungen'),
      content: SingleChildScrollView(
        child: Column(children: [
          CheckboxListTile(
            value: analytics,
            onChanged: (v) => setState(() => analytics = v ?? false),
            title: const Text('App-Nutzungsstatistik senden (freiwillig)'),
          ),
          CheckboxListTile(
            value: newsletter,
            onChanged: (v) => setState(() => newsletter = v ?? false),
            title: const Text('Produkt-News per E-Mail erhalten'),
          ),
          CheckboxListTile(
            value: sensitives,
            onChanged: (v) => setState(() => sensitives = v ?? false),
            title: const Text('Religion/Politik im Profil anzeigen'),
          ),
          CheckboxListTile(
            value: location,
            onChanged: (v) => setState(() => location = v ?? false),
            title: const Text('Standort-basiertes Matching aktivieren'),
          ),
        ]),
      ),
      actions: [
        TextButton(
          onPressed: () {
            widget.onSave(
              analytics: analytics,
              newsletter: newsletter,
              sensitives: sensitives,
              location: location,
            );
            Navigator.pop(context);
          },
          child: const Text('Speichern'),
        ),
      ],
    );
  }
}
```

---

### 2 · PHP-Endpoint zum Speichern der Consents

_(angepasster Pfad: `userdata.consents.*`)_

```php
// POST /v1/consent
$input = json_decode(file_get_contents('php://input'), true);

$fields = [
  'analytics', 'newsletter', 'sensitives', 'location'
];

$set = [];
foreach ($fields as $f) {
    if (array_key_exists($f, $input)) {
        $set["userdata.consents.$f"] = [
            'granted'   => (bool)$input[$f],
            'timestamp' => new MongoDB\BSON\UTCDateTime()
        ];
    }
}
$users->updateOne(['_id' => $userId], ['$set' => $set]);
```

---

### 3 · Privacy-Policy Textbausteine (neue Abschnitte)

```md
### 3. Konto & Chat (Art. 6 Abs. 1 lit. b DSGVO)
Wir verarbeiten Ihre Registrierungs-, Profil- und Chat-Daten, um den vertraglich geschuldeten Dienst bereitzustellen.

### 4. Freiwillige Angaben besonderer Kategorien (Art. 9 Abs. 2 a DSGVO)
Sie können auf Wunsch Religion oder politische Ansichten in Ihrem Profil anzeigen. Diese Daten werden erst veröffentlicht, wenn Sie der Anzeige ausdrücklich zustimmen und können jederzeit in den Einstellungen ausgeblendet werden.

### 5. Standortbasiertes Matching (Einwilligung, Art. 6 Abs. 1 lit. a)
Mit Ihrer Zustimmung nutzen wir Ihren ungefähren Standort, um Profile in Ihrer Nähe vorzuschlagen. Sie können die Funktion jederzeit deaktivieren.

### 6. Sicherheits-Logs (Art. 6 Abs. 1 lit. f)
Zur Abwehr von Spam und Angriffen speichern wir verkürzte IP-Adressen und Geräte-Fingerprints. Unser berechtigtes Interesse überwiegt Ihre Rechte, da die Daten nur pseudonym vorliegen und spätestens 30 Tage nach Account-Löschung gelöscht werden.

### 7. Nutzungsanalyse (nur mit Einwilligung, Art. 6 Abs. 1 lit. a)
Wenn Sie zustimmen, erfassen wir anonymisierte Nutzungsstatistiken ...

### 8. Newsletter (Einwilligung, Art. 6 Abs. 1 lit. a)
E-Mails mit Updates erhalten Sie nur ...
```

---

### Nächste Schritte

1. **UI-Flows testen:** alle vier Checkboxen (Analytics, Newsletter, Sensitives, Location).
    
2. **RoPA ergänzen:** neue Consent-Flags (`userdata.consents.*`) + Religion/Politik (Art. 9) + Location (Geodaten).
    
3. **Balancing-Test dokumentieren** für Security-Logs & ggf. Crash-Logs (lit. f).
    
4. **Data-Safety-Form** in Google Play:
    
    - Datenkategorien: „Geolocation“, „Sensitive info (Political or Religious beliefs)“ (nur wenn Opt-in gesetzt).
        
    - Zweck: „App functionality“, „Analytics“ (opt.).
        

Damit ist dein Art-6/Art-9-Mapping, Consent-Flow, API und Privacy-Policy vollständig auf das neue Daten­schema abgestimmt. Sag Bescheid, falls du noch eine exportierbare CSV, Unit-Tests oder ein Beispiel für den location-Consent auf iOS brauchst!
# old 
Hier siehst du ein **kompaktes Praxis-Muster**, wie du Art. 6 DSGVO in deinem Projekt „übersetzen“ kannst – inklusive UI-Element, Back-End-Feldnamen und Platzhalter-Text für die Privacy-Policy. Kopiere einfach die Teile, die du brauchst.

| Verarbeitungsvorgang                            | Betroffene Datenfelder                      | Rechtsgrundlage (Art. 6 Abs. 1)        | WIE du sie erfüllst                                                  | Was im Code/DB landet                                    |
| ----------------------------------------------- | ------------------------------------------- | -------------------------------------- | -------------------------------------------------------------------- | -------------------------------------------------------- |
| **Kern-Account & Chat**                         | username, e-mail, PW-Hash, Nachrichten      | lit. b (Vertrag)                       | *Nutzungsbedingungen akzeptieren* im Onboarding (Checkbox)           | kein Consent-Flag nötig                                  |
| **Device- & Abuse-Logs**                        | IP-Log, fingerprint_hash, verification_code | lit. f (berechtigtes Interesse)        | Privacy-Policy-Abschnitt „Sicherheit“ + interne Interessenabwägung   | kein Opt-in – aber Löschfrist & Kürzung der IP speichern |
| **Crash- & Nutzungsanalyse (Firebase, Sentry)** | pseudonyme Nutzer-ID, Device-Infos          | lit. a (Einwilligung) **ODER** lit. f* | Vor dem 1. Event: Consent-Dialog („Darfst du Nutzungsdaten senden?“) | `consents.analytics.granted = true/false` + timestamp    |
| **Marketing-/Newsletter-Mails**                 | e-mail + Tracking-Events                    | lit. a (Einwilligung)                  | Separate Checkbox **abgewählt**; Double-Opt-In-Mail                  | `consents.newsletter.granted = true/false` + timestamp   |
| **Rechnungs-/Steuerdaten (Abo-Zahlung)**        | Name, Adresse, Stripe-ID                    | lit. c (gesetzliche Pflicht)           | Verweis auf HGB/AO in Privacy Policy                                 | Daten in separater „finance“-Collection, 10 J. Retention |
*Viele Start-ups wählen für Crash-Logs lieber **lit. f** (Sicherstellung der Stabilität). Dann musst du im Balancing-Test (intern) begründen, warum kein Opt-in nötig ist und wie du Anonymisierung sicherstellst.


1 Flutter-UI – Consent-Dialog
```
class ConsentDialog extends StatelessWidget {
  final void Function(bool analyticsOk, bool newsletterOk) onSave;
  const ConsentDialog({required this.onSave, super.key});

  @override
  Widget build(BuildContext context) {
    bool analytics = false;
    bool newsletter = false;

    return AlertDialog(
      title: const Text('Datenschutz-Einstellungen'),
      content: Column(mainAxisSize: MainAxisSize.min, children: [
        CheckboxListTile(
          value: analytics,
          onChanged: (v) => analytics = v ?? false,
          title: const Text('App-Nutzungsstatistik senden (freiwillig)'),
        ),
        CheckboxListTile(
          value: newsletter,
          onChanged: (v) => newsletter = v ?? false,
          title: const Text('Produkt-News per E-Mail erhalten'),
        ),
      ]),
      actions: [
        TextButton(
          onPressed: () {
            onSave(analytics, newsletter);
            Navigator.pop(context);
          },
          child: const Text('Speichern'),
        )
      ],
    );
  }
}

```
2 PHP-API für Consent-Speicher
```
// POST /v1/consent
$input = json_decode(file_get_contents('php://input'), true);

$set = [];
if (array_key_exists('analytics', $input)) {
    $set['user_data.consents.analytics'] = [
        'granted'   => (bool)$input['analytics'],
        'timestamp' => new MongoDB\BSON\UTCDateTime()
    ];
}
if (array_key_exists('newsletter', $input)) {
    $set['user_data.consents.newsletter'] = [
        'granted'   => (bool)$input['newsletter'],
        'timestamp' => new MongoDB\BSON\UTCDateTime()
    ];
}
$users->updateOne(['_id' => $userId], ['$set' => $set]);

```

3 Textbausteine für deine Privacy-Policy
```
### 3. Konto & Chat (Art. 6 Abs. 1 lit. b DSGVO)
Wir verarbeiten Ihre Registrierungs- und Chat-Daten, um den vertraglich geschuldeten Dienst bereitzustellen.

### 4. Sicherheits-Logs (Art. 6 Abs. 1 lit. f)
Zur Abwehr von Spam und Angriffen speichern wir verkürzte IP-Adressen und Geräte-Fingerprints. Unser berechtigtes Interesse überwiegt, da …

### 5. Nutzungsanalyse (nur mit Einwilligung, Art. 6 Abs. 1 lit. a)
Wenn Sie zustimmen, senden wir anonymisierte Nutzungsstatistiken an Google Firebase …

### 6. Newsletter (Einwilligung, Art. 6 Abs. 1 lit. a)
Sie erhalten E-Mails mit Updates nur, wenn Sie dies ausdrücklich aktiviert haben. Ein Widerruf ist jederzeit möglich.

```
### Nächste Schritte

1. **Consents live testen** (Opt-in/Opt-out Pfad).
    
2. **VVT/ROPA** um die neuen Felder `consents.*` ergänzen (Legal Basis = lit. a, Retention = bis Widerruf + 30 Tage).
    
3. **Prüfe Dritt-SDKs** – alles, was eine eigene IP-Adresse sieht, gehört entweder in _lit. a_ (Consent) oder braucht einen starken Balancing-Test für _lit. f_.

