### Was ist „Crashlytics“ – und brauchst du es überhaupt?

|Punkt|Kurz erklärt|
|---|---|
|**Dienst**|**Firebase Crashlytics** ist Googles Echtzeit-Crash-Reporter: Er sammelt Stack-Traces, Gerätemodell, OS-Version, App-Version, anonyme Installation-IDs u. ä., gruppiert Abstürze und zeigt sie im Firebase-Dashboard.([developers.google.com](https://developers.google.com/ad-manager/mobile-ads-sdk/ios/crashlytics?utm_source=chatgpt.com "Logging Ad Response ID with Firebase Crashlytics"))|
|**Teil von Firebase**|Auch wenn der Name „Crashlytics“ noch einzeln kursiert (früher Fabric), läuft es seit 2019 **vollständig unter Firebase**. Wer Crashlytics einbindet, nutzt also _doch_ Firebase-Backend-Dienste.|
|**Gespeicherte Daten & Aufbewahrung**|Google behält Stack-Traces und zugehörige IDs bis zu **90 Tagen**.([firebase.google.com](https://firebase.google.com/support/privacy?utm_source=chatgpt.com "Privacy and Security in Firebase - Google"))|
|**Rechtslage (GDPR/DSGVO)**|Die Absturz­daten sind **pseudonyme personenbezogene Daten**. Du musst daher: 1. einen **DPA** mit Google akzeptieren (im Firebase-Konto), 2. Crashlytics im Privacy-Policy & Play-“Data Safety” offenlegen, 3. entweder eine **Einwilligung** einholen — **oder** per Interessen­abwägung (Art. 6 (1) f) belegen, dass Crash-Logs zwingend für die App-Stabilität sind.|
|**Opt-in-Technik**|In Flutter: `FirebaseCrashlytics.instance.setCrashlyticsCollectionEnabled(false);` und erst nach Zustimmung auf `true` setzen.|
|**Alternativen**|_Selbst hostbar_ → [Sentry](https://docs.sentry.io) (on-prem), _EU-Region_ → Bugsnag EU, Instabug EU. Alle brauchen ebenfalls DPA, aber manche bieten dedizierte EU-Server.|
|**Brauche ich es?**|Bei wenigen Beta-Nutzer*innen kannst du auch ohne externes Crash-SDK starten (lokale Logs, Feedback-E-Mail). Steigt die Nutzerzahl, wird ein Dashboard fast unverzichtbar – dann rechtzeitig Consent-Flow oder Balancing-Test implementieren.|

---

#### Wie das konkret in deinem Projekt aussehen könnte

1. **Wenn du _kein_ Crashlytics möchtest**
    
    - Entferne die Abhängigkeit `flutterfire_crashlytics` (oder ähnliches) aus `pubspec.yaml`.
        
    - Halte Exceptions lokal (z. B. per `FlutterError.onError` → REST-Endpoint in deinem Backend).
        
2. **Wenn du Crashlytics behalten willst**
    
    |To-do|Code / Konsole|
    |---|---|
    |DPA akzeptieren|Firebase Console → ⚙️ > „Kontoeinstellungen“ > „Datenschutz & Sicherheit“ > EU-Datenverarbeitungsbedingungen|
    |Collection standardmäßig OFF|`dart<br>await Firebase.initializeApp();<br>await FirebaseCrashlytics.instance.setCrashlyticsCollectionEnabled(false); // vor Consent`|
    |Consent-Dialog erweitern|Checkbox „App-Stabilitätsdaten an Google senden“. Bei Opt-in → `setCrashlyticsCollectionEnabled(true)`.|
    |Privacy-Policy-Abschnitt|„Crash-Reports via Firebase Crashlytics (Google Ireland Ltd., 90 Tage, Serverstandort EU/USA, DPA + SCC) – nur nach Einwilligung/ berechtigtem Interesse.“|
    |Play-Console Data-Safety|Sammel-Kategorie „App activity > Crash logs“, Zweck „App functionality“ **und/oder** „Analytics“, retention „≤ 90 Tage“, „Data encrypted in transit: Yes“, „User deletion request respected: Yes“.|
    
3. **Balancing-Test statt Consent** (Option)
    
    - Begründe intern: _„Ohne Crash-Logs können wir schwerwiegende Fehler nicht zeitnah beheben; Betroffenheit der Nutzer gering, da nur pseudonyme IDs.“_
        
    - Biete trotzdem eine **Opt-out-Switch** in „Einstellungen → Datenschutz“.
        

---

### TL;DR

- **Crashlytics** = Firebase-Crash-Reporting-Service.
    
- DSGVO-relevant, weil er pseudonyme Nutzer-IDs + Stack-Traces an Google sendet.
    
- Du hast zwei Wege:
    
    - **Ausschalten / gar nicht einbauen** → kein weiterer Aufwand.
        
    - **Einbauen** → DPA + Privacy-Policy + Data-Safety + Consent _oder_ gut begründetes berechtigtes Interesse.
        

Wenn du dir erst einmal den ganzen Firebase-Rattenschwanz sparen willst, starte ohne Crashlytics und setze auf lokales Error-Logging; du kannst das SDK später hinzufügen, sobald die Investoren- oder Nutzerskalierung es erfordert.