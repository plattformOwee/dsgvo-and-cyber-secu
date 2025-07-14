Lets focus back on what the main mission of this chat is: DSGVO compliance. Here are prior documents we made for this purpose :

"

| **Dateninventur & RoPA** [[RoPA woven]] (Record of Processing Activities) | Nachvollziehbarkeit (§30 DSGVO) | Erstelle ein Sheet: _Welche Felder_, _wo_ gespeichert (Mongo-Collection), _warum_, _Retention_ |

| ------------------------------------------------------------------------------------------- | --------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

| [[Rechtsgrundlage festlegen]] | Art. 6 DSGVO | “Vertrag” (Nutzungs-Abo) reicht meistens; für optionale Mails/Analytics zusätzliche Einwilligung |

| [[Privacy Policy und Impressum]] | Transparenz-Pflicht (Art. 12–14) | Verlinkt im Onboarding + Play-Store “Data Safety”-Formular ([termly.io](https://termly.io/resources/checklists/gdpr-requirements/?utm_source=chatgpt.com "5 Step GDPR Requirements Checklist For All Businesses - Termly")) |

| **Einwilligungs-Flow für optionale SDKs** (Ads, Crashlytics, Firebase etc.) [[crashlytics]] | EU-User-Consent-Policy von Google | Nutze Google UMP-SDK oder eigenes Dialog-Widget ([developers.google.com](https://developers.google.com/admob/android/privacy/gdpr?utm_source=chatgpt.com "GDPR IAB support \| Android - Google for Developers")) |

| [[User-Rights-API]] (Auskunft, Löschung, Portabilität) [[]] | Art. 15-20 | Simple REST-Endpoints: /me/export, /me/delete. Löschen in Mongo = Remove + Backup-Flag |

| [[Datenminimierung & Security]] | Art. 5 (1) c + Art. 32 | Speichere wirklich nur E-Mail + salted Hash; TLS überall; Mongo-IP-whitelisting |

| [[Auftragsverarbeitung]] | Art. 28 | Hoster, Crash-SDKs, Mail-Provider → schließe DPAs (Standardvertragsklauseln, wenn non-EU) |

| [[Breach-Process]] | Art. 33 | Lege eine 72-Stunden-Checkliste + _critical incident Slack channel_ an |

| **Optional: DPO bestellen** | Pflicht erst bei regelmäßiger _umfangreicher_ Überwachung | Kann i. d. R. entfallen bei < 250 MA und kleinem Profil-Service |"

[[Rechtsgrundlage festlegen]]:

"

## Praxis-Mapping Art. 6 / Art. 9 DSGVO – **neues Datenmodell**

| Verarbeitungsvorgang | Betroffene Datenfelder* | Rechtsgrundlage (Art. 6 / 9) | WIE du sie erfüllst (UX / Dokument) | Was im Code / DB landet |

| ------------------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------ | ---------------------------------------------------------------------------------- | -------------------------------------------------- |

| **Kern-Account & Chat** | `profile.topSection.name`, `userdata.email`, Chat-Nachrichten | **lit. b** (Vertrag) | ToS + Privacy-Checkbox im Onboarding | kein Consent-Flag nötig |

| **Profilbilder & Eisbrecher / Fragen** | `profile.profileImages[]`, `profile.elements[].content.answer` | **lit. b** | Upload-Button bzw. Freitextfelder → Teil der Kern­funktionen | – |

| **Freiwillige „InfoBubbles“ – Religion, Politik** | `infoBubbles.religion`, `infoBubbles.politics` | **Art. 9 (2)(a)** + **Art. 6 (1)(a)** (Einwill.) | Extra-Schalter „Diese sensiblen Angaben veröffentlichen“ (Opt-in, default **off**) | `consents.profile_sensitives.granted = true/false` |

| **Standort + Radius** (Nah-Matching) | `search_filter.location_radius.location / radius` | **lit. a** (Einwill.) | OS-Location-Prompt + App-Dialog „Standort­basiertes Matching aktivieren?“ | `consents.location.granted = true/false` |

| **Device- & Abuse-Logs** | IP-Logs (Server), `userdata.fingerprint_hash`, `verification_code` | **lit. f** (berechtigtes Interesse) | Absatz „Sicherheit“ in Privacy-Policy + interner Balancing-Test | kein Opt-in; Rotations-/Löschfristen |

| **Crash- & Nutzungsanalyse** (Firebase / Sentry) | pseudonyme Instanz-ID, Stacktrace, Device-Infos | **lit. a** *oder* **lit. f*** | Vor 1. Event: Consent-Dialog-Checkbox **„App-Statistiken senden“** | `consents.analytics.granted` + timestamp |

| **Marketing / Newsletter** | `userdata.email`, Öffnungs-Tracking | **lit. a** | Getrennte Checkbox (abgewählt), Double-Opt-In-Mail | `consents.newsletter.granted` + timestamp |

| **Zahlungs- / Rechnungsdaten** (In-App-Abo außer Store) | Name, Adresse, Stripe-ID, Transaktions-ID | **lit. c** (gesetzl. Pflicht) | Verweis auf HGB/AO im Privacy-Policy; Datensatz in `finance`-Collection | 10 J. Aufbewahrung |

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

bool sensitives = false; // Religion/Politik freigeben

bool location = false; // Standort-Matching

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

'granted' => (bool)$input[$f],

'timestamp' => new MongoDB\BSON\UTCDateTime()

];

}

}

$users->updateOne(['_id' => $userId], ['$set' => $set]);

"

Bis jetzt haben wir:

"

consents: {

location: { granted: true, timestamp: ISODate('2025-07-11T17:48:13.947Z') },

microphone: { granted: true, timestamp: ISODate('2025-07-11T17:48:13.947Z') },

notifications: {

granted: false,

timestamp: ISODate('2025-07-11T17:48:13.947Z')

}

}"
````


es fehlt in der also:
"
| **Freiwillige „InfoBubbles“ – Religion, Politik** | `infoBubbles.religion`, `infoBubbles.politics` | **Art. 9 (2)(a)** + **Art. 6 (1)(a)** (Einwill.) | Extra-Schalter „Diese sensiblen Angaben veröffentlichen“ (Opt-in, default **off**) | `consents.profile_sensitives.granted = true/false` |

| ------------------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------ | ---------------------------------------------------------------------------------- | -------------------------------------------------- |"

"