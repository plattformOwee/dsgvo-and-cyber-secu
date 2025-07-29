### Zu Punkt 1: Ratenbegrenzung & Genehmigungsprozess

Ihre Schlussfolgerung ist im Prinzip richtig, braucht aber eine wichtige Differenzierung. Nicht *jede* Aktion benötigt den *vollen* Zwei-Admin-Genehmigungsprozess. Das wäre für den Betrieb unpraktikabel. Wir sollten die Sicherheitsmaßnahmen nach Kritikalität abstufen:

**Stufe 1: Ratenbegrenzung (Für fast alle Aktionen)**
*   **Was?** Schützt vor Spam und automatisiertem Missbrauch (Brute-Force, DDoS).
*   **Wo anwenden?**
    *   Login-Versuche (`login.php`)
    *   Registrierung (`signup.php`)
    *   Code-Verifizierung (`verify_code.php`)
    *   Alle User-Rights-API-Endpunkte (`request_me.php`, `delete_me.php` etc.)
    *   Nachricht senden (`sendMessage.php`) -> Verhindert Chat-Spam.
    *   Profil-Reaktionen (`react_profile.php`) -> Verhindert Swipe-Bots.
*   **Umsetzung:** `has_exceeded_rate_limit()` und `log_rate_limit_attempt()` wie besprochen.

**Stufe 2: Protokollierung (Audit Trail ohne Genehmigung)**
*   **Was?** Schreibt eine Aktion in ein Protokoll (z.B. die `pending_actions` Tabelle, aber mit Status `completed` oder in ein separates Log), ohne auf eine Genehmigung zu warten. Dient der reinen Nachvollziehbarkeit.
*   **Wo anwenden?**
    *   Passwort ändern
    *   E-Mail-Adresse ändern
*   **Zweck:** Sie können sehen, wann ein Nutzer kritische Account-Daten geändert hat.

**Stufe 3: Zwei-Admin-Genehmigung (Nur für das " nukleare" Option)**
*   **Was?** Der volle `request_action_approval` Workflow, der eine zweite Person erfordert.
*   **Wo anwenden?**
    *   **Endgültige, irreversible Löschung** von Nutzerdaten durch einen **Admin** im Backend.
    *   **Bestätigung der vom Nutzer beantragten Löschung** (als Sicherheits-Check).
    *   Massenhafte Änderungen oder Löschungen, die von einem Admin-Skript ausgeführt werden.
*   **Zweck:** Schutz vor katastrophalen Fehlern oder internem Missbrauch.

**Fazit zu Punkt 1:** Ja, alle kritischen Aktionen gehen in den `request_action_approval` Prozess. Viele andere werden "nur" per Rate-Limiting geschützt.

---

### Zu Punkt 2: Ablehnung von Löschanträgen

Ja, das ist zulässig, aber nur unter sehr spezifischen Umständen. Sie lehnen nicht das *Recht* des Nutzers ab, sondern **pausieren die Ausführung** der Löschung, um einer unmittelbaren Bedrohung nachzugehen.

Die Rechtsgrundlage dafür ist Ihr **berechtigtes Interesse (Art. 6 Abs. 1 lit. f DSGVO)**, die Sicherheit Ihrer Plattform und die Daten Ihrer anderen Nutzer zu schützen.

**Der korrekte Prozess wäre:**
1.  Ein Admin bemerkt die Anomalie (z.B. 1000 Löschanträge in 5 Minuten).
2.  Er lehnt die Genehmigung im Admin-Panel ab und gibt als Grund an: "Aussetzung wegen Verdachts auf einen koordinierten Angriff. Untersuchung eingeleitet."
3.  Das Team untersucht die Ursache.
4.  Sobald das Problem gelöst ist (z.B. der Angriff wurde abgewehrt, ein Bug wurde gefixt), werden die legitimen Löschanträge (falls vorhanden) erneut zur Genehmigung vorgelegt und ausgeführt.

Sie können eine Löschung also **vorübergehend aussetzen**, um die Sicherheit zu gewährleisten, aber nicht permanent verweigern.

---

### Zu Punkt 3: Der neue `delete_me` Workflow

Ihr Verständnis ist **exakt richtig**. Hier ist der verfeinerte Workflow, Schritt für Schritt:

1.  **Nutzer klickt auf "Konto löschen"** in der App und authentifiziert sich (Passwort/Biometrie).
2.  Der `delete_me.php`-Endpunkt wird aufgerufen.
3.  **Sofortige Aktionen im Backend:**
    a. `request_action_approval()` wird aufgerufen. Ein neuer Eintrag wird in `pending_actions` mit `status: 'pending'` erstellt. Die `justification` ist fest kodiert: "Nutzerantrag über die App (Art. 17 DSGVO)".
    b. Eine E-Mail wird an den Nutzer gesendet: "Wir haben Ihren Löschantrag erhalten. Wenn Sie das nicht waren, kontaktieren Sie den Support."
4.  **Im Admin-Panel:** Der neue Antrag erscheint in der Liste der zu genehmigenden Aktionen. Es gibt *keine* feste 5-Tage-Frist, sondern die Admins bearbeiten dies "unverzüglich" (was einige Tage dauern kann).
5.  **Admin genehmigt:** Ein Admin prüft den Antrag auf Plausibilität und klickt auf "Genehmigen".
6.  **Der Status in `pending_actions` wechselt zu `approved`**.
7.  **Der Cronjob:** Ein periodisch laufender Cronjob sucht nach Einträgen mit `status: 'approved'`.
8.  **Finale Ausführung:** Für jeden gefundenen Eintrag führt der Cronjob die endgültige Aktion aus: Er löscht die Nutzerdaten oder kennzeichnet sie für die anonymisierte Aufbewahrung (je nach Ihrer Policy). Danach wird der Status auf `completed` gesetzt.

Dieser Prozess ist robust, sicher und DSGVO-konform.

---

### Zu Punkt 4: Admin-Panel

**Genau.** Sie benötigen zwingend ein sicheres Admin-Panel, das nur für autorisierte Mitarbeiter zugänglich ist. Dieses Panel ist die Benutzeroberfläche für den gesamten Genehmigungsprozess. Es muss mindestens Folgendes anzeigen und ermöglichen:
*   Eine Liste aller Anträge mit dem Status `pending`.
*   Für jeden Antrag: Wer hat ihn wann mit welcher Begründung gestellt?
*   Buttons für "Genehmigen" und "Ablehnen".
*   Bei "Ablehnen" ein Pflichtfeld zur Eingabe einer Begründung.
*   Ein Archiv aller abgeschlossenen (`approved`, `rejected`, `completed`) Aktionen (das ist Ihr Audit Trail).

---

### Ihre letzte Frage: Welche Prozesse benötigen wirklich eine "Justification" vom NUTZER?

Die Antwort ist einfach und wichtig:

**Praktisch keiner, wenn der Nutzer seine gesetzlichen DSGVO-Rechte ausübt.**

Ein Nutzer muss **niemals begründen**, warum er sein Recht auf Auskunft, Berichtigung, Übertragbarkeit oder Löschung in Anspruch nehmen möchte. Die Ausübung dieser Rechte ist bedingungslos.

Eine "Begründung" oder ein Freitextfeld für den Nutzer ist in anderen Kontexten sinnvoll, die **keine Ausübung von DSGVO-Rechten** sind, zum Beispiel:

*   **Einen anderen Nutzer melden:** HIER ist ein Feld "Bitte beschreiben Sie den Vorfall" absolut notwendig.
*   **Support kontaktieren:** Ein Feld "Wie können wir Ihnen helfen?" ist der Kern einer Support-Anfrage.
*   **Feedback zur App geben:** Ein Feld "Was können wir verbessern?" ist wünschenswert.

Merken Sie sich die Faustregel: Fordern Sie niemals eine Begründung an, wenn ein Nutzer ein Recht ausübt, das ihm per Gesetz zusteht.

# antwort

Task1: Bitte erstelle das admin panel in einer main.dart file
- filter für verschiedene kategorien an pending actions 
- login page verbunden mit backend wo ich manuell was rein mache 
- mit endpoints für alle nötigen aktionen die die selbe datenbank benutzen aber collection "admin_users"

Task2: 
- die backend scripte dazu
- 