Hier siehst du ein **kompaktes Praxis-Muster**, wie du Art. 6 DSGVO in deinem Projekt „übersetzen“ kannst – inklusive UI-Element, Back-End-Feldnamen und Platzhalter-Text für die Privacy-Policy. Kopiere einfach die Teile, die du brauchst.

| Verarbeitungsvorgang                              | Betroffene Datenfelder                            | Rechtsgrundlage (Art. 6 Abs. 1) | WIE du sie erfüllst                                                            | Was im Code/DB landet                                    |
|---------------------------------------------------|---------------------------------------------------|---------------------------------|--------------------------------------------------------------------------------|----------------------------------------------------------|
| **Kern-Account & Chat**                           | username, e-mail, PW-Hash, Nachrichten            | lit. b (Vertrag)                | *Nutzungsbedingungen akzeptieren* im Onboarding (Checkbox)                    | kein Consent-Flag nötig                                  |
| **Device- & Abuse-Logs**                          | IP-Log, fingerprint_hash, verification_code       | lit. f (berechtigtes Interesse) | Privacy-Policy-Abschnitt „Sicherheit“ + interne Interessenabwägung            | kein Opt-in – aber Löschfrist & Kürzung der IP speichern |
| **Crash- & Nutzungsanalyse (Firebase, Sentry)**   | pseudonyme Nutzer-ID, Device-Infos                | lit. a (Einwilligung) **ODER** lit. f* | Vor dem 1. Event: Consent-Dialog („Darfst du Nutzungsdaten senden?“)          | `consents.analytics.granted = true/false` + timestamp    |
| **Marketing-/Newsletter-Mails**                   | e-mail + Tracking-Events                          | lit. a (Einwilligung)           | Separate Checkbox **abgewählt**; Double-Opt-In-Mail                           | `consents.newsletter.granted = true/false` + timestamp   |
| **Rechnungs-/Steuerdaten (Abo-Zahlung)**          | Name, Adresse, Stripe-ID                          | lit. c (gesetzliche Pflicht)    | Verweis auf HGB/AO in Privacy Policy                                          | Daten in separater „finance“-Collection, 10 J. Retention |
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

